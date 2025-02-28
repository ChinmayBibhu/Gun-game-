import pygame
import math
from pygame.locals import *

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 1200, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Shooting Range")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREY = (50, 50, 50)

# Target details
TARGET_CENTER = (WIDTH // 2, HEIGHT // 3)
TARGET_RINGS = [160, 130, 100, 70, 40]  # Correctly spaced rings
TARGET_COLORS = [BLACK, WHITE, BLACK, WHITE, BLACK]
TARGET_SCORES = [50, 40, 30, 20, 10]

# Gun details
GUN_Y = HEIGHT - 100  # Gun at bottom of screen
GUN_WIDTH, GUN_HEIGHT = 80, 40

class Bullet:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = -15  # Moves upwards
        self.active = True

bullets = []
hit_marks = []
mouse_x = WIDTH // 2
clock = pygame.time.Clock()
running = True

while running:
    screen.fill(WHITE)

    # Handle input
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        if event.type == MOUSEMOTION:
            mouse_x = event.pos[0]
        if event.type == KEYDOWN and event.key == K_SPACE:
            bullets.append(Bullet(mouse_x, GUN_Y - 10))

    # Move bullets
    for bullet in bullets[:]:
        bullet.y += bullet.speed
        if bullet.y < TARGET_CENTER[1] + TARGET_RINGS[0]:  # Check collision
            for i, radius in enumerate(TARGET_RINGS):
                if math.hypot(bullet.x - TARGET_CENTER[0], bullet.y - TARGET_CENTER[1]) <= radius:
                    hit_marks.append((bullet.x, bullet.y, TARGET_SCORES[i]))
                    bullets.remove(bullet)
                    break

    # Draw target
    for i, radius in enumerate(TARGET_RINGS):
        pygame.draw.circle(screen, TARGET_COLORS[i], TARGET_CENTER, radius)

    # Draw bullets
    for bullet in bullets:
        pygame.draw.circle(screen, BLACK, (bullet.x, bullet.y), 5)

    # Draw gun (just a simple horizontal block)
    pygame.draw.rect(screen, GREY, (mouse_x - GUN_WIDTH // 2, GUN_Y, GUN_WIDTH, GUN_HEIGHT))

    # Draw hit markers
    font = pygame.font.SysFont(None, 36)
    for x, y, score in hit_marks:
        pygame.draw.circle(screen, RED, (x, y), 5)
        text = font.render(str(score), True, RED)
        screen.blit(text, (x + 10, y - 10))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
