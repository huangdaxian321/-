import pygame
import random

# Initialize Pygame
pygame.init()

# Screen size
WIDTH, HEIGHT = 600, 600
CELL_SIZE = 20
TOTAL_TIME = 120  # seconds

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GREEN = (0, 200, 0)
RED = (220, 30, 30)
BLUE = (50, 150, 255)
GRAY = (120, 120, 120)
BACKGROUND = (30, 30, 30)

# Create screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Complex Snake Game')
clock = pygame.time.Clock()

# Font
font = pygame.font.SysFont('arial', 32)
big_font = pygame.font.SysFont('arial', 48, bold=True)


def draw_text(text, color, x, y, center=False, big=False):
    f = big_font if big else font
    surface = f.render(text, True, color)
    rect = surface.get_rect()
    if center:
        rect.center = (x, y)
    else:
        rect.topleft = (x, y)
    screen.blit(surface, rect)


class Snake:
    def __init__(self, positions, direction, color):
        self.positions = positions
        self.direction = direction
        self.color = color
        self.grow = False

    def move(self, wrap=False):
        x, y = self.positions[0]
        if self.direction == 'UP':
            y -= CELL_SIZE
        elif self.direction == 'DOWN':
            y += CELL_SIZE
        elif self.direction == 'LEFT':
            x -= CELL_SIZE
        elif self.direction == 'RIGHT':
            x += CELL_SIZE

        if wrap:
            x %= WIDTH
            y %= HEIGHT
        else:
            if x < 0 or x >= WIDTH or y < 0 or y >= HEIGHT:
                return False  # Hit the wall

        new_head = (x, y)
        if new_head in self.positions:
            return False  # Collision with self

        self.positions.insert(0, new_head)
        if not self.grow:
            self.positions.pop()
        else:
            self.grow = False
        return True

    def change_direction(self, new_dir):
        opposites = {
            'UP': 'DOWN', 'DOWN': 'UP', 'LEFT': 'RIGHT', 'RIGHT': 'LEFT'
        }
        if opposites[new_dir] != self.direction:
            self.direction = new_dir

    def draw(self):
        for pos in self.positions:
            rect = pygame.Rect(pos[0], pos[1], CELL_SIZE, CELL_SIZE)
            pygame.draw.rect(screen, self.color, rect, border_radius=4)


class Food:
    def __init__(self):
        self.position = self.random_position()

    def random_position(self):
        while True:
            pos = (
                random.randrange(0, WIDTH, CELL_SIZE),
                random.randrange(0, HEIGHT, CELL_SIZE),
            )
            if (
                pos not in player_snake.positions
                and pos not in ai_snake.positions
                and pos not in obstacles
            ):
                return pos

    def draw(self):
        rect = pygame.Rect(self.position[0], self.position[1], CELL_SIZE, CELL_SIZE)
        pygame.draw.rect(screen, RED, rect, border_radius=4)


def generate_obstacles(count):
    obs = []
    for _ in range(count):
        while True:
            pos = (random.randrange(0, WIDTH, CELL_SIZE),
                   random.randrange(0, HEIGHT, CELL_SIZE))
            if (
                pos not in player_snake.positions
                and pos not in ai_snake.positions
                and pos not in obs
                and pos != food.position
            ):
                obs.append(pos)
                break
    return obs


def respawn_ai():
    """Respawn the AI snake with length 1 at a random empty location."""
    global ai_snake, ai_score
    while True:
        pos = (
            random.randrange(0, WIDTH, CELL_SIZE),
            random.randrange(0, HEIGHT, CELL_SIZE),
        )
        if (
            pos not in player_snake.positions
            and pos not in obstacles
            and pos != food.position
        ):
            ai_snake = Snake([pos], random.choice(['UP', 'DOWN', 'LEFT', 'RIGHT']), (255, 200, 0))
            ai_score = 0
            break


def game_over_screen(message):
    screen.fill(RED)
    pygame.display.flip()
    pygame.time.delay(500)
    screen.fill(BACKGROUND)
    draw_text(message, WHITE, WIDTH // 2, HEIGHT // 2 - 40, center=True, big=True)
    draw_text('Press R to play again or Q to quit', WHITE, WIDTH // 2, HEIGHT // 2 + 20, center=True)
    pygame.display.flip()
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    return True
                elif event.key == pygame.K_q:
                    return False
        clock.tick(15)


def run_game():
    global player_snake, ai_snake, obstacles, food, player_score, ai_score, level, speed

    player_snake = Snake([(WIDTH // 3, HEIGHT // 2)], 'RIGHT', GREEN)
    ai_snake = Snake([(2 * WIDTH // 3, HEIGHT // 2)], 'LEFT', (255, 200, 0))
    obstacles = []
    food = Food()
    player_score = 0
    ai_score = 0
    level = 1
    speed = 10
    start_time = pygame.time.get_ticks()

    running = True
    reason = 'Game Over'
    while running:
        clock.tick(speed)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    player_snake.change_direction('UP')
                elif event.key == pygame.K_DOWN:
                    player_snake.change_direction('DOWN')
                elif event.key == pygame.K_LEFT:
                    player_snake.change_direction('LEFT')
                elif event.key == pygame.K_RIGHT:
                    player_snake.change_direction('RIGHT')

        elapsed = (pygame.time.get_ticks() - start_time) / 1000
        remaining = int(max(0, TOTAL_TIME - elapsed))
        if remaining <= 0:
            reason = 'Time up'
            break

        # 玩家蛇移动
        if not player_snake.move():
            reason = 'Crashed'
            break
        if (
            player_snake.positions[0] in obstacles
            or player_snake.positions[0] in ai_snake.positions
        ):
            reason = 'Crashed'
            break

        # AI蛇逻辑
        def ai_choose_direction():
            directions = ['UP', 'DOWN', 'LEFT', 'RIGHT']
            best = None
            best_dist = float('inf')
            for d in directions:
                x, y = ai_snake.positions[0]
                if d == 'UP':
                    y -= CELL_SIZE
                elif d == 'DOWN':
                    y += CELL_SIZE
                elif d == 'LEFT':
                    x -= CELL_SIZE
                elif d == 'RIGHT':
                    x += CELL_SIZE
                pos = (x, y)
                if (
                    pos in ai_snake.positions
                    or pos in obstacles
                    or pos in player_snake.positions
                    or x < 0
                    or x >= WIDTH
                    or y < 0
                    or y >= HEIGHT
                ):
                    continue
                dist = abs(pos[0] - food.position[0]) + abs(pos[1] - food.position[1])
                if dist < best_dist:
                    best_dist = dist
                    best = d
            if best is None:
                best = random.choice(directions)
            # introduce 5% chance to pick a random direction instead
            if random.random() < 0.05:
                best = random.choice(directions)
            ai_snake.direction = best

        ai_choose_direction()
        if not ai_snake.move():
            respawn_ai()
        elif ai_snake.positions[0] in obstacles or ai_snake.positions[0] in player_snake.positions:
            respawn_ai()
        if ai_snake.positions[0] == food.position:
            ai_snake.grow = True
            ai_score += 1
            food = Food()

        # 玩家吃到食物
        if player_snake.positions[0] == food.position:
            player_snake.grow = True
            player_score += 1
            if player_score % 5 == 0:
                level += 1
                speed += 2
                obstacles.extend(generate_obstacles(2))
            food = Food()

        # 渲染
        screen.fill(BACKGROUND)
        player_snake.draw()
        ai_snake.draw()
        food.draw()
        for pos in obstacles:
            rect = pygame.Rect(pos[0], pos[1], CELL_SIZE, CELL_SIZE)
            pygame.draw.rect(screen, GRAY, rect, border_radius=4)

        time_text = f"{remaining // 60:02}:{remaining % 60:02}"
        draw_text(
            f'Player: {player_score}  AI: {ai_score}  Level: {level}  Time: {time_text}',
            WHITE,
            10,
            10,
        )
        pygame.display.flip()

    return game_over_screen(reason)



def main():
    running = True
    while running:
        running = run_game()   # 这里run_game()最后return的是game_over_screen的返回值
    pygame.quit()

main()



