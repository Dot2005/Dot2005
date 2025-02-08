import pygame
import math

# Initialize Pygame
pygame.init()

# Set screen dimensions
screen_width, screen_height = 800, 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Basketball Game")

# Define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
RED = (255, 0, 0)

# Define player and goal positions
player_pos = [400, 300]  # Starting position of the player (center of screen)
player_jump = False
player_velocity = 0
gravity = 0.5
jump_strength = -10
player_speed = 5  # Horizontal speed of the player

goal_positions = [(100, 100), (700, 100)]  # Example goals at the two ends

# Camera parameters
camera_x, camera_y = 0, 0
camera_zoom = 1  # Camera zoom level (optional)

# Ball class to simulate shooting
class Ball:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.radius = 10
        self.velocity_x = 0
        self.velocity_y = 0
        self.is_shooting = False

    def shoot(self, angle, power):
        self.velocity_x = power * math.cos(angle)
        self.velocity_y = power * math.sin(angle)
        self.is_shooting = True

    def update(self):
        if self.is_shooting:
            # Apply physics to simulate arc
            self.x += self.velocity_x
            self.y += self.velocity_y
            self.velocity_y += gravity  # Gravity pull
            
            # If the ball reaches the ground or out of bounds, reset
            if self.y >= 300:  # Check if ball hits the ground
                self.y = 300
                self.is_shooting = False
                self.reset_ball()

    def reset_ball(self):
        # Optionally reset the ball position
        self.x = player_pos[0] + 25
        self.y = player_pos[1] + 25
        self.velocity_x = 0
        self.velocity_y = 0

    def draw(self, screen):
        pygame.draw.circle(screen, RED, (int(self.x - camera_x), int(self.y - camera_y)), self.radius)

# Player class
class Player:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 50
        self.is_jumping = False
        self.velocity = 0

    def move(self, dx, dy):
        self.x += dx
        self.y += dy

    def jump(self):
        if not self.is_jumping:
            self.velocity = jump_strength
            self.is_jumping = True

    def update(self):
        if self.is_jumping:
            self.velocity += gravity  # Apply gravity
            self.y += self.velocity
            if self.y >= 300:  # Player hits the ground
                self.y = 300
                self.is_jumping = False
                self.velocity = 0

    def draw(self, screen):
        pygame.draw.rect(screen, GREEN, (self.x - camera_x, self.y - camera_y, self.width, self.height))

# Function to calculate the distance between two points
def calculate_distance(p1, p2):
    return math.sqrt((p2[0] - p1[0])**2 + (p2[1] - p1[1])**2)

# Camera follow logic with smoothing
def follow_ball(ball):
    global camera_x, camera_y
    # Smooth camera transition using linear interpolation
    camera_x += (ball.x - screen_width / 2 - camera_x) * 0.1
    camera_y += (ball.y - screen_height / 2 - camera_y) * 0.1

def follow_goal(goal_pos):
    global camera_x, camera_y
    # Smooth camera transition using linear interpolation
    camera_x += (goal_pos[0] - screen_width / 2 - camera_x) * 0.1
    camera_y += (goal_pos[1] - screen_height / 2 - camera_y) * 0.1

# Main game loop
def main():
    global player_pos, player_jump, camera_x, camera_y

    player = Player(player_pos[0], player_pos[1])
    ball = Ball(player.x + 25, player.y + 25)

    clock = pygame.time.Clock()
    running = True

    while running:
        screen.fill(WHITE)
        player.update()
        ball.update()
        
        # Check for events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    player.jump()  # Jump when space is pressed

        # Handle player movement (left and right)
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            player.move(-player_speed, 0)
        if keys[pygame.K_RIGHT]:
            player.move(player_speed, 0)

        # If player jumps, automatically calculate the closest goal and shoot the ball
        if player.is_jumping and not ball.is_shooting:
            # Calculate distance to each goal
            distances = [calculate_distance((player.x, player.y), goal) for goal in goal_positions]
            closest_goal_index = distances.index(min(distances))
            closest_goal = goal_positions[closest_goal_index]
            
            # Lock camera to follow the closest goal
            follow_goal(closest_goal)

            # Automatically shoot the ball towards the closest goal
            shoot_angle = math.atan2(closest_goal[1] - player.y, closest_goal[0] - player.x)
            shoot_power = 25  # High arc power
            ball.shoot(shoot_angle, shoot_power)

        # If ball is shooting, follow the ball instead of the goal
        if ball.is_shooting:
            follow_ball(ball)

        # Draw player, ball, and goals
        player.draw(screen)
        ball.draw(screen)
        for goal in goal_positions:
            pygame.draw.circle(screen, BLACK, (goal[0] - camera_x, goal[1] - camera_y), 20)  # Draw goals

        # Update screen
        pygame.display.flip()

        # Frame rate
        clock.tick(60)

# Run the game
if __name__ == "__main__":
    main()
