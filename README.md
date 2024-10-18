import pygame
import sys
import random

# Inicializar Pygame
pygame.init()

# Configurar la pantalla
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Física en Aventura")

# Cargar imágenes
player_image = pygame.image.load('player.png')
platform_image = pygame.image.load('platform.png')
trampoline_image = pygame.image.load('trampoline.png')
water_image = pygame.image.load('water.png')
background_image = pygame.image.load('background.png')

# Cargar sonidos
jump_sound = pygame.mixer.Sound('jump.wav')
hit_sound = pygame.mixer.Sound('hit.wav')
game_over_sound = pygame.mixer.Sound('game_over.wav')

# Colores
BLUE = (0, 0, 255)

# Clase para el jugador
class Player:
    def __init__(self):
        self.pos = [100, 500]
        self.size = 50
        self.velocity = 5
        self.gravity = 0.5
        self.jump_strength = 10
        self.is_jumping = False
        self.vertical_velocity = 0
        self.jump_time = 0

    def move(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.pos[0] -= self.velocity
        if keys[pygame.K_RIGHT]:
            self.pos[0] += self.velocity

        # Limitar el movimiento del jugador a la pantalla
        self.pos[0] = max(0, min(WIDTH - self.size, self.pos[0]))

    def jump(self):
        if not self.is_jumping:
            keys = pygame.key.get_pressed()
            if keys[pygame.K_SPACE]:
                self.is_jumping = True
                self.vertical_velocity = -self.jump_strength
                jump_sound.play()
                self.jump_time = 0
        else:
            self.jump_time += 1
            if self.jump_time < 15:
                self.vertical_velocity += self.gravity
            else:
                self.vertical_velocity += self.gravity * 1.5
            
            self.pos[1] += self.vertical_velocity
            if self.pos[1] >= 500:
                self.pos[1] = 500
                self.is_jumping = False
                self.vertical_velocity = 0
                self.jump_time = 0

    def draw(self, screen):
        screen.blit(player_image, self.pos)

# Clase para obstáculos
class Obstacle:
    def __init__(self, x, y, width, height):
        self.rect = pygame.Rect(x, y, width, height)

    def draw(self, screen):
        pygame.draw.rect(screen, BLUE, self.rect)

# Función para crear plataformas
def create_platforms():
    return [
        [0, 550, 800, 50],
        [300, 400, 200, 20]
    ]

# Generar obstáculos aleatorios
def generate_obstacles(num):
    obstacles = []
    while len(obstacles) < num:
        width = random.randint(50, 100)
        height = random.randint(20, 50)
        x = random.randint(0, WIDTH - width)
        y = random.randint(0, HEIGHT - height - 50)
        new_obstacle = Obstacle(x, y, width, height)
        if not any(obs.rect.colliderect(new_obstacle.rect) for obs in obstacles):
            obstacles.append(new_obstacle)
    return obstacles

# Reiniciar juego
def reset_game():
    global player, obstacles, score
    player = Player()
    obstacles = generate_obstacles(5)
    score = 0

# Inicializar juego
player = Player()
platforms = create_platforms()
trampoline = [350, 370, 100, 20]
water_area = [0, 500, 800, 50]
obstacles = generate_obstacles(5)
score = 0
font = pygame.font.Font(None, 36)

# Bucle principal
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    player.move()
    player.jump()

    # Colisiones con plataformas
    for plat in platforms:
        if (plat[0] < player.pos[0] < plat[0] + plat[2]) and (plat[1] < player.pos[1] + player.size < plat[1] + plat[3]):
            player.pos[1] = plat[1] - player.size
            player.is_jumping = False
            player.vertical_velocity = 0
            player.jump_time = 0

    # Colisión con el trampolín
    if (trampoline[0] < player.pos[0] < trampoline[0] + trampoline[2]) and (trampoline[1] < player.pos[1] + player.size < trampoline[1] + trampoline[3]):
        player.pos[1] = trampoline[1] - player.size
        player.is_jumping = True
        player.vertical_velocity = -player.jump_strength * 1.5
        jump_sound.play()

    # Colisión con el agua
    if (water_area[0] < player.pos[0] < water_area[0] + water_area[2]) and (water_area[1] < player.pos[1] + player.size < water_area[1] + water_area[3]):
        player.vertical_velocity += player.gravity * 0.1
        player.pos[1] += player.vertical_velocity

    # Colisión con obstáculos
    for obs in obstacles:
        if obs.rect.colliderect(pygame.Rect(player.pos[0], player.pos[1], player.size, player.size)):
            hit_sound.play()
            player.pos[1] = obs.rect.top - player.size
            player.is_jumping = False
            player.vertical_velocity = 0
            player.jump_time = 0

    # Comprobar si el jugador cae al agua o se sale de la pantalla
    if player.pos[1] > HEIGHT:
        game_over_sound.play()
        reset_game()  # Reiniciar juego si cae

    # Limpiar la pantalla
    screen.blit(background_image, (0, 0))

    # Dibujar todo
    for plat in platforms:
        screen.blit(platform_image, (plat[0], plat[1]))
    player.draw(screen)
    for obs in obstacles:
        obs.draw(screen)
    screen.blit(trampoline_image, (trampoline[0], trampoline[1]))
    screen.blit(water_image, (water_area[0], water_area[1]))

    # Aumentar puntuación con el tiempo
    score += 1

    # Dibujar puntuación
    score_text = font.render(f"Puntuación: {score}", True, (255, 255, 255))
    screen.blit(score_text, (10, 10))

    # Actualizar pantalla
    pygame.display.flip()
    pygame.time.delay(30)
