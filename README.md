import pygame
import math
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 1200, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("3D Shooting Range")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREY = (100, 100, 100)

# 3D Projection parameters
FOV = 60
CAMERA_DISTANCE = 500

# Target ring sizes
TARGET_RINGS = [160, 130, 100, 70, 40]  # Properly spaced rings
TARGET_COLORS = [BLACK, WHITE, BLACK, WHITE, BLACK]  # Alternating colors
TARGET_SCORES = [50, 40, 30, 20, 10]  # Scoring

class GameObject:
    def __init__(self):
        self.position = [0, 0, 0]
        self.rotation = [0, 0, 0]

class Gun(GameObject):
    def __init__(self):
        super().__init__()
        self.position = [0, 0, CAMERA_DISTANCE - 200]  # Place gun in foreground
        self.rotation = [0, 0, 0]  # Pitch, Yaw, Roll

class Target(GameObject):
    def __init__(self):
        super().__init__()
        self.position = [0, 0, CAMERA_DISTANCE + 800]  # Place target far away

class Bullet:
    def __init__(self, position, rotation):
        self.position = position[:]
        self.rotation = rotation[:]
        self.velocity = [
            math.sin(math.radians(rotation[1])) * math.cos(math.radians(rotation[0])),
            -math.sin(math.radians(rotation[0])),
            math.cos(math.radians(rotation[1])) * math.cos(math.radians(rotation[0]))
        ]
        self.speed = 40
        self.active = True

def project_3d_to_2d(point):
    """ Project a 3D point to 2D screen space """
    if point[2] <= 0:
        return None
    factor = CAMERA_DISTANCE / (CAMERA_DISTANCE + point[2])
    x = int(WIDTH / 2 + point[0] * factor)
    y = int(HEIGHT / 2 - point[1] * factor)
    return (x, y)

def draw_target(target):
    """ Draw target with properly spaced rings """
    center = project_3d_to_2d(target.position)
    if not center:
        return
    for i, radius in enumerate(TARGET_RINGS):
        screen_radius = int(radius * CAMERA_DISTANCE / (CAMERA_DISTANCE + target.position[2]))
        pygame.draw.circle(screen, TARGET_COLORS[i], center, screen_radius)
        if i < len(TARGET_RINGS) - 1:
            pygame.draw.circle(screen, BLACK, center, screen_radius, 3)

def draw_gun(gun):
    """ Draw gun as a simple rectangle to simulate distance """
    gun_pos = project_3d_to_2d(gun.position)
    if not gun_pos:
        return
    gun_width = 80
    gun_height = 40
    pygame.draw.rect(screen, GREY, (gun_pos[0] - gun_width // 2, gun_pos[1] - gun_height // 2, gun_width, gun_height))
    pygame.draw.rect(screen, BLACK, (gun_pos[0] - gun_width // 2, gun_pos[1] - gun_height // 2, gun_width, gun_height), 3)

def check_target_hit(bullet, target):
    """ Check if bullet hits the target and return score """
    relative_pos = [bullet.position[0] - target.position[0], bullet.position[1] - target.position[1]]
    distance = math.hypot(relative_pos[0], relative_pos[1])
    
    for i, radius in enumerate(TARGET_RINGS):
        if distance <= radius:
            return TARGET_SCORES[i], relative_pos
    return 0, None

# Initialize objects
gun = Gun()
target = Target()
bullets = []
hit_marks = []

mouse_sensitivity = 0.2
clock = pygame.time.Clock()
running = True

while running:
    screen.fill(WHITE)
    
    # Handle input
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        if event.type == KEYDOWN and event.key == K_SPACE:
            bullets.append(Bullet(gun.position, gun.rotation))

    # Mouse movement
    mouse_dx, mouse_dy = pygame.mouse.get_rel()
    gun.rotation[0] += mouse_dy * mouse_sensitivity
    gun.rotation[1] += mouse_dx * mouse_sensitivity
    gun.rotation[0] = max(min(gun.rotation[0], 45), -45)  # Clamp vertical rotation

    # Update bullets
    for bullet in bullets[:]:
        if bullet.active:
            bullet.position[0] += bullet.velocity[0] * bullet.speed
            bullet.position[1] += bullet.velocity[1] * bullet.speed
            bullet.position[2] += bullet.velocity[2] * bullet.speed
            
            # Check if bullet hits the target
            if bullet.position[2] >= target.position[2]:
                score, impact_pos = check_target_hit(bullet, target)
                if score > 0:
                    hit_marks.append({'position': impact_pos, 'score': score, 'timer': 60})
                bullets.remove(bullet)

    # Draw elements
    draw_target(target)
    draw_gun(gun)
    
    # Draw hit marks
    for hit in hit_marks[:]:
        if hit['timer'] > 0:
            impact_pos = [target.position[0] + hit['position'][0], target.position[1] + hit['position'][1], target.position[2]]
            impact_screen_pos = project_3d_to_2d(impact_pos)
            if impact_screen_pos:
                pygame.draw.circle(screen, RED, impact_screen_pos, 5)
                font = pygame.font.SysFont(None, 36)
                text = font.render(str(hit['score']), True, RED)
                screen.blit(text, (impact_screen_pos[0] + 10, impact_screen_pos[1] - 10))
                hit['timer'] -= 1
        else:
            hit_marks.remove(hit)
    
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
