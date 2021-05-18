import copy
import pygame


class Board:
    # создание поля
    def __init__(self, width, height, left=10, top=10, cell_size=29):
        self.width = width
        self.height = height
        self.board = [[0] * width for _ in range(height)]
        # значения по умолчанию
        self.left = left
        self.top = top
        self.cell_size = cell_size
        self.set_view(left, top, cell_size)

    # настройка внешнего вида
    def set_view(self, left, top, cell_size):
        pass

    # отрисовка на холсте
    def render(self, screen):
        pass

    # возвращает координаты клетки в виде кортежа
    def get_cell(self, mouse_pos):
        cell_x = (mouse_pos[0] - self.left) // self.cell_size
        cell_y = (mouse_pos[1] - self.top) // self.cell_size
        if cell_x < 0 or cell_y < 0 or cell_x >= self.width or cell_y >= self.height:
            return None
        return cell_x, cell_y

    # изменение поля(клеток) по полученным координатам
    def on_click(self, cell_coords):
        self.board[cell_coords[1]][cell_coords[0]] = (self.board[cell_coords[1]][cell_coords[0]] + 1) % 2

    # "диспетчер"
    def get_click(self, mouse_pos):
        cell = self.get_cell(mouse_pos)
        if cell:
            self.on_click(cell)
        else:
            print(cell)


class Life(Board):
    def __init__(self, width, height, left=10, top=10, cell_size=29):
        super().__init__(width, height, left, top, cell_size)

    def on_click(self, cell_coords):
        self.board[cell_coords[1]][cell_coords[0]] = (self.board[cell_coords[1]][cell_coords[0]] + 1) % 2

    def render(self, screen):
        colors = [pygame.Color('black'), pygame.Color('green')]
        for y in range(self.height):
            for x in range(self.width):
                pygame.draw.rect(screen,
                                 colors[self.board[y][x]],
                                 [(x * self.cell_size + self.left, y * self.cell_size + self.top),
                                  (self.cell_size, self.cell_size)])
                pygame.draw.rect(screen,
                                 pygame.Color('white'),
                                 [(x * self.cell_size + self.left, y * self.cell_size + self.top),
                                  (self.cell_size, self.cell_size)], 1)

    def next_move(self):
        # сохраняем поле
        tmp_board = copy.deepcopy(self.board)
        # пересчитываем
        for y in range(self.height):
            for x in range(self.width):
                # сумма окружающих клеток
                s = 0
                for dy in range(-1, 2):
                    for dx in range(-1, 2):
                        if x + dx < 0 or x + dx >= self.width or y + dy < 0 or y + dy >= self.height:
                            continue
                        s += self.board[y + dy][x + dx]
                s -= self.board[y][x]
                if s == 3:
                    tmp_board[y][x] = 1
                elif s < 2 or s > 3:
                    tmp_board[y][x] = 0
        # обновляем поле
        self.board = copy.deepcopy(tmp_board)


def main():
    pygame.init()
    size = 900, 900
    screen = pygame.display.set_mode(size)
    clock = pygame.time.Clock()
    pygame.display.set_caption('Игра «Жизнь»')

    board = Life(30, 30, 10, 10, 29)

    # Включено ли обновление поля
    time_on = False
    ticks = 0
    speed = 10

    running = True

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                board.get_click(event.pos)
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE or \
                    event.type == pygame.MOUSEBUTTONDOWN and event.button == 3:
                time_on = not time_on
            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 4:
                speed += 1
            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 5:
                speed -= 1

        screen.fill((0, 0, 0))
        board.render(screen)
        if ticks >= speed:
            if time_on:
                board.next_move()
            ticks = 0
        pygame.display.flip()
        clock.tick(100)
        ticks += 1
    pygame.quit()


if __name__ == '__main__':
    main()
