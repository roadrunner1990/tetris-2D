import pygame
import random

pygame.init()

# Constants
SCREEN_WIDTH = 300
SCREEN_HEIGHT = 600
GRID_SIZE = 30
GRID_WIDTH = SCREEN_WIDTH // GRID_SIZE
GRID_HEIGHT = SCREEN_HEIGHT // GRID_SIZE

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
CYAN = (0, 255, 255)
MAGENTA = (255, 0, 255)

# Shapes
SHAPES = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[0, 1, 0], [1, 1, 1]],  # T
    [[0, 1, 1], [1, 1, 0]],  # S
    [[1, 1, 0], [0, 1, 1]],  # Z
    [[1, 1, 1], [1, 0, 0]],  # J
    [[1, 1, 1], [0, 0, 1]]  # L
]

COLORS = [RED, GREEN, BLUE, YELLOW, CYAN, MAGENTA, WHITE]

class Tetris:
    def __init__(self):
        self.screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
        pygame.display.set_caption('Tetris')
        self.clock = pygame.time.Clock()
        self.grid = [[BLACK for _ in range(GRID_WIDTH)] for _ in range(GRID_HEIGHT)]
        self.current_shape = self.get_new_shape()
        self.next_shape = self.get_new_shape()
        self.score = 0
        self.game_over = False

    def get_new_shape(self):
        shape = random.choice(SHAPES)
        color = random.choice(COLORS)
        return {'shape': shape, 'color': color, 'x': GRID_WIDTH // 2 - len(shape[0]) // 2, 'y': 0}

    def rotate_shape(self):
        self.current_shape['shape'] = [list(row) for row in zip(*self.current_shape['shape'][::-1])]

    def check_collision(self):
        shape = self.current_shape['shape']
        for y, row in enumerate(shape):
            for x, cell in enumerate(row):
                if cell and (self.current_shape['y'] + y >= GRID_HEIGHT or self.grid[self.current_shape['y'] + y][self.current_shape['x'] + x] != BLACK):
                    return True
        return False

    def lock_shape(self):
        shape = self.current_shape['shape']
        for y, row in enumerate(shape):
            for x, cell in enumerate(row):
                if cell:
                    self.grid[self.current_shape['y'] + y][self.current_shape['x'] + x] = self.current_shape['color']
        self.clear_lines()
        self.current_shape = self.next_shape
        self.next_shape = self.get_new_shape()
        if self.check_collision():
            self.game_over = True

    def clear_lines(self):
        new_grid = [row for row in self.grid if BLACK in row]
        lines_cleared = len(self.grid) - len(new_grid)
        self.score += lines_cleared ** 2
        self.grid = [[BLACK for _ in range(GRID_WIDTH)] for _ in range(lines_cleared)] + new_grid

    def draw_grid(self):
        for y in range(GRID_HEIGHT):
            for x in range(GRID_WIDTH):
                pygame.draw.rect(self.screen, self.grid[y][x], pygame.Rect(x * GRID_SIZE, y * GRID_SIZE, GRID_SIZE, GRID_SIZE))

    def draw_shape(self):
        shape = self.current_shape['shape']
        color = self.current_shape['color']
        for y, row in enumerate(shape):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(self.screen, color, pygame.Rect((self.current_shape['x'] + x) * GRID_SIZE, (self.current_shape['y'] + y) * GRID_SIZE, GRID_SIZE, GRID_SIZE))

    def draw_next_shape(self):
        shape = self.next_shape['shape']
        color = self.next_shape['color']
        for y, row in enumerate(shape):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(self.screen, color, pygame.Rect((GRID_WIDTH + x) * GRID_SIZE // 2, y * GRID_SIZE // 2, GRID_SIZE // 2, GRID_SIZE // 2))

    def run(self):
        while not self.game_over:
            self.screen.fill(BLACK)
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.game_over = True
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        self.current_shape['x'] -= 1
                        if self.check_collision():
                            self.current_shape['x'] += 1
                    elif event.key == pygame.K_RIGHT:
                        self.current_shape['x'] += 1
                        if self.check_collision():
                            self.current_shape['x'] -= 1
                    elif event.key == pygame.K_DOWN:
                        self.current_shape['y'] += 1
                        if self.check_collision():
                            self.current_shape['y'] -= 1
                            self.lock_shape()
                    elif event.key == pygame.K_UP:
                        self.rotate_shape()
                        if self.check_collision():
                            self.rotate_shape()
                            self.rotate_shape()
                            self.rotate_shape()

            self.current_shape['y'] += 1
            if self.check_collision():
                self.current_shape['y'] -= 1
                self.lock_shape()

            self.draw_grid()
            self.draw_shape()
            self.draw_next_shape()
            pygame.display.flip()
            self.clock.tick(10)

if __name__ == '__main__':
    game = Tetris()
    game.run()
    pygame.quit()
