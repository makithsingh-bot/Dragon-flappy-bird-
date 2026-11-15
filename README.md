# Dragon-flappy-bird-

import pygame
import random
import sys

pygame.init()

screen = pygame.display.set_mode((0, 0))
pygame.display.set_caption("Dragon Flappy Infinity")

W, H = screen.get_size()
clock = pygame.time.Clock()

WHITE = (255, 255, 255)
GREEN = (35, 190, 65)
DARK = (10, 105, 35)
BLUE = (85, 190, 245)
BROWN = (160, 105, 55)
YELLOW = (255, 220, 70)

font = pygame.font.Font(None, 38)
bigfont = pygame.font.Font(None, 65)

ground = int(H * 0.88)

# Dragon
player = pygame.Rect(80, H//2, 55, 45)
vy = 0

# Pipe configuration
PIPE_W = max(65, int(W * 0.18))
GAP = max(125, int(H * 0.19))

# थोड़ी ज्यादा दूरी
PIPE_DISTANCE = max(190, int(W * 0.62))

pipes = []

score = 0
best = 0
speed = 4
frame = 0
state = "menu"

def reset_game():
    global pipes, score, speed, frame, vy

    player.x = int(W * 0.20)
    player.y = H // 2

    pipes = []
    score = 0
    speed = 4
    frame = 0
    vy = 0

    add_pipe(W + 80)

def add_pipe(x):
    margin = int(H * 0.08)

    gap_center = random.randint(
        margin + GAP//2,
        ground - margin - GAP//2
    )

    pipes.append({
        "x": float(x),
        "center": gap_center,
        "passed": False
    })

def draw_text(msg, f, color, x, y):
    img = f.render(msg, True, color)
    screen.blit(img, img.get_rect(center=(x, y)))

def draw_button(rect, msg, color):
    pygame.draw.rect(
        screen, color, rect, border_radius=15
    )
    draw_text(
        msg, font, WHITE,
        rect.centerx, rect.centery
    )

def draw_background():
    screen.fill(BLUE)

    # Clouds
    for i in range(4):
        x = (i * W//3 - frame) % (W + 80)

        pygame.draw.circle(
            screen, WHITE, (x, H//5), 18
        )
        pygame.draw.circle(
            screen, WHITE, (x+20, H//5-8), 22
        )
        pygame.draw.circle(
            screen, WHITE, (x+40, H//5), 17
        )

    # Ground
    pygame.draw.rect(
        screen, BROWN,
        (0, ground, W, H-ground)
    )

    pygame.draw.rect(
        screen, GREEN,
        (0, ground, W, 8)
    )

def draw_pipe(x, y, width, height, top):
    if height <= 0:
        return

    x = int(x)
    y = int(y)

    # Thick pipe
    pygame.draw.rect(
        screen, DARK,
        (x, y, width, height)
    )

    pygame.draw.rect(
        screen, GREEN,
        (x+8, y, width-16, height)
    )

    # Pipe cap
    cap_h = 24

    if top:
        cap_y = y + height - cap_h
    else:
        cap_y = y

    pygame.draw.rect(
        screen, DARK,
        (x-9, cap_y, width+18, cap_h)
    )

    pygame.draw.rect(
        screen, GREEN,
        (x-5, cap_y+4, width+10, cap_h-8)
    )

def draw_dragon():
    x, y = player.x, player.y

    # Tail
    pygame.draw.polygon(
        screen, GREEN,
        [(x+5,y+25),(x-17,y+10),(x+5,y+38)]
    )

    # Body
    pygame.draw.ellipse(
        screen, GREEN,
        (x,y+10,45,32)
    )

    # Wing
    pygame.draw.polygon(
        screen, DARK,
        [(x+20,y+25),(x+10,y-10),(x+42,y+20)]
    )

    # Head
    pygame.draw.circle(
        screen, GREEN,
        (x+48,y+14), 17
    )

    # Horn
    pygame.draw.polygon(
        screen, YELLOW,
        [(x+40,y+2),(x+36,y-10),(x+48,y+2)]
    )

    # Eye
    pygame.draw.circle(
        screen, WHITE,
        (x+53,y+9), 5
    )

    pygame.draw.circle(
        screen, DARK,
        (x+54,y+9), 2
    )

def flap():
    global vy
    if state == "play":
        vy = -8

def update_game():
    global score, speed, frame, vy, state, best

    frame += 1

    # Dragon physics
    vy += 0.42
    vy = min(vy, 9)
    player.y += int(vy)

    if player.top < 0:
        player.top = 0
        vy = 0

    if player.bottom >= ground:
        player.bottom = ground
        state = "over"

    # Speed increase
    speed = min(4 + score // 15, 10)

    # Pipes with slightly increased distance
    if not pipes or pipes[-1]["x"] < W - PIPE_DISTANCE:
        add_pipe(W + 20)

    for p in pipes:
        p["x"] -= speed

        gap_top = p["center"] - GAP//2
        gap_bottom = p["center"] + GAP//2

        # Top pipe
        draw_pipe(
            p["x"], 0,
            PIPE_W, gap_top, True
        )

        # Bottom pipe
        draw_pipe(
            p["x"], gap_bottom,
            PIPE_W, ground-gap_bottom, False
        )

        top_rect = pygame.Rect(
            int(p["x"]), 0,
            PIPE_W, gap_top
        )

        bottom_rect = pygame.Rect(
            int(p["x"]), gap_bottom,
            PIPE_W, ground-gap_bottom
        )

        hitbox = player.inflate(-12, -10)

        if hitbox.colliderect(top_rect):
            state = "over"

        if hitbox.colliderect(bottom_rect):
            state = "over"

        # Score
        if not p["passed"] and p["x"] + PIPE_W < player.x:
            p["passed"] = True
            score += 1
            best = max(best, score)

    pipes[:] = [
        p for p in pipes
        if p["x"] + PIPE_W > -20
    ]

    draw_dragon()

    draw_text(
        "SCORE: " + str(score),
        font, WHITE, 100, 40
    )

    draw_text(
        "BEST: " + str(best),
        font, WHITE, W-100, 40
    )

def main():
    global state

    running = True

    while running:
        clock.tick(60)

        for event in pygame.event.get():

            if event.type == pygame.QUIT:
                running = False

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    running = False
                if event.key == pygame.K_SPACE:
                    flap()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if state == "menu":
                    state = "play"
                    reset_game()

                elif state == "over":
                    reset_game()
                    state = "play"

                else:
                    flap()

            if event.type == pygame.FINGERDOWN:
                if state == "menu":
                    state = "play"
                    reset_game()

                elif state == "over":
                    reset_game()
                    state = "play"

                else:
                    flap()

        draw_background()

        if state == "menu":
            draw_text(
                "DRAGON FLAPPY",
                bigfont, WHITE,
                W//2, H//2-100
            )

            draw_dragon()

            draw_button(
                pygame.Rect(W//2-110,H//2,220,75),
                "START GAME",
                GREEN
            )

            draw_text(
                "TAP TO FLY",
                font, WHITE,
                W//2, H//2+120
            )

        elif state == "play":
            update_game()

        elif state == "over":
            draw_dragon()

            draw_text(
                "GAME OVER",
                bigfont, WHITE,
                W//2, H//2-60
            )

            draw_text(
                "SCORE: "+str(score),
                font, WHITE,
                W//2, H//2+5
            )

            draw_button(
                pygame.Rect(W//2-110,H//2+65,220,70),
                "RESTART",
                GREEN
            )

        pygame.display.flip()

    pygame.quit()
    sys.exit()

reset_game()
