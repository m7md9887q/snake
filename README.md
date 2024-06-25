from kivy.app import App
from kivy.uix.widget import Widget
from kivy.clock import Clock
import pygame
import random
import sys

class PygameWidget(Widget):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.pygame_initialized = False
        Clock.schedule_once(self.init_pygame, 0)

    def init_pygame(self, dt):
        pygame.init()
        self.screen = pygame.display.set_mode((800, 600))
        pygame.display.set_caption("Snake Game")
        self.snake_size = 20
        self.snake_speed = 7
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont(None, 35)
        self.running = True
        self.die_on_collision = self.game_menu()
        self.game_loop()

    def game_menu(self):
        self.screen.fill((144, 238, 144))
        self.message("Snake Game", (0, 0, 0), 800 / 3, 600 / 4)
        self.message("Choose Game Mode:", (0, 0, 0), 800 / 3, 600 / 3)
        option1_rect = pygame.Rect(800 / 3, 600 / 2, 200, 50)
        option2_rect = pygame.Rect(800 / 3, 600 / 2 + 70, 200, 50)
        pygame.draw.rect(self.screen, (0, 100, 0), option1_rect)
        pygame.draw.rect(self.screen, (0, 100, 0), option2_rect)
        self.message("Option 1: Die on Wall Collision (1)", (0, 0, 0), 800 / 3 + 10, 600 / 2 + 10)
        self.message("Option 2: Warp Through Walls (2)", (0, 0, 0), 800 / 3 + 10, 600 / 2 + 80)
        pygame.display.update()
        while True:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_1:
                        return True
                    elif event.key == pygame.K_2:
                        return False

    def game_loop(self):
        game_over = False
        game_close = False
        score = 0
        x1 = 800 / 2
        y1 = 600 / 2
        x1_change = 0
        y1_change = 0
        snake_List = []
        Length_of_snake = 1
        foodx = round(random.randrange(0, 800 - self.snake_size) / self.snake_size) * self.snake_size
        foody = round(random.randrange(0, 600 - self.snake_size) / self.snake_size) * self.snake_size

        while not game_over:
            while game_close:
                self.screen.fill((144, 238, 144))
                self.message("You Lost! Press Q-Quit or C-Back to Menu", (255, 0, 0), 800 / 6, 600 / 3)
                self.message("Score: " + str(score), (0, 0, 0), 800 / 6, 600 / 3 + 50)
                pygame.display.update()
                for event in pygame.event.get():
                    if event.type == pygame.KEYDOWN:
                        if event.key == pygame.K_q:
                            pygame.quit()
                            sys.exit()
                        if event.key == pygame.K_c:
                            return

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        x1_change = -self.snake_size
                        y1_change = 0
                    elif event.key == pygame.K_RIGHT:
                        x1_change = self.snake_size
                        y1_change = 0
                    elif event.key == pygame.K_UP:
                        y1_change = -self.snake_size
                        x1_change = 0
                    elif event.key == pygame.K_DOWN:
                        y1_change = self.snake_size
                        x1_change = 0

            if x1 >= 800 or x1 < 0 or y1 >= 600 or y1 < 0:
                if self.die_on_collision:
                    game_close = True
                else:
                    if x1 >= 800:
                        x1 = 0
                    elif x1 < 0:
                        x1 = 800 - self.snake_size
                    elif y1 >= 600:
                        y1 = 0
                    elif y1 < 0:
                        y1 = 600 - self.snake_size

            x1 += x1_change
            y1 += y1_change

            self.draw_grid()

            pygame.draw.rect(self.screen, (255, 0, 0), [foodx, foody, self.snake_size, self.snake_size])
            snake_Head = []
            snake_Head.append(x1)
            snake_Head.append(y1)
            snake_List.append(snake_Head)
            if len(snake_List) > Length_of_snake:
                del snake_List[0]

            for segment in snake_List[:-1]:
                if segment == snake_Head:
                    game_close = True

            for segment in snake_List:
                pygame.draw.rect(self.screen, (0, 255, 0), [segment[0], segment[1], self.snake_size, self.snake_size])

            self.display_score(score)

            pygame.display.update()

            if x1 == foodx and y1 == foody:
                score += 1
                foodx = round(random.randrange(0, 800 - self.snake_size) / self.snake_size) * self.snake_size
                foody = round(random.randrange(0, 600 - self.snake_size) / self.snake_size) * self.snake_size
                Length_of_snake += 1

            self.clock.tick(self.snake_speed)

    def draw_grid(self):
        for x in range(0, 800, self.snake_size):
            for y in range(0, 600, self.snake_size):
                rect = pygame.Rect(x, y, self.snake_size, self.snake_size)
                if (x + y) // self.snake_size % 2 == 0:
                    pygame.draw.rect(self.screen, (0, 0, 0), rect)
                else:
                    pygame.draw.rect(self.screen, (0, 0, 0), rect)

    def message(self, msg, color, x, y):
        mesg = self.font.render(msg, True, color)
        self.screen.blit(mesg, [x, y])

    def display_score(self, score):
        score_text = self.font.render("Score: " + str(score), True, (0, 0, 0))
        self.screen.blit(score_text, [0, 0])

class SnakeApp(App):
    def build(self):
        return PygameWidget()

if __name__ == '__main__':
    SnakeApp().run()
