from tkinter import *
import random

# Collision function
def rects_collide(r1, r2):
    x1, y1, w1, h1 = r1
    x2, y2, w2, h2 = r2
    return not (x1 + w1 < x2 or x1 > x2 + w2 or
                y1 + h1 < y2 or y1 > y2 + h2)


# Game class
class Game:
    SCREEN_WIDTH = 1100
    SCREEN_HEIGHT = 600

    def __init__(self):
        self.root = Tk()
        self.root.title("Engineer Runner (OOP)")
        self.canvas = Canvas(self.root, width=self.SCREEN_WIDTH, height=self.SCREEN_HEIGHT)
        self.canvas.pack()

        # Load images
        self.images = {}
        self.load_images()

        # Adjust background
        bg_height = self.images["CITY_BG"].height()
        if bg_height >= self.SCREEN_HEIGHT:
            self.BG_Y = 0
        else:
            self.BG_Y = self.SCREEN_HEIGHT - bg_height

        # Ground level
        self.GROUND_Y = self.SCREEN_HEIGHT - 50

        # Game state
        self.x_pos_bg = 0
        self.game_speed = 15
        self.points = 0
        self.high_score = 0
        self.state = "menu"

        # Game objects
        self.obstacles = []
        self.player = None

        # Keyboard binding
        self.root.bind("<KeyPress>", self.on_key_press)
        self.root.bind("<KeyRelease>", self.on_key_release)

        # Start menu
        self.show_menu()

        self.root.mainloop()

    # Load images
    def load_images(self):
        def load(name, key):
            img = PhotoImage(file=name)
            self.images[key] = img

        load("GirlRun1.png", "GIRL_RUN1")
        load("GirlRun2.png", "GIRL_RUN2")
        load("GirlJump.png", "GIRL_JUMP")
        load("Barrier.png", "BARRIER")
        load("CityBackground.png", "CITY_BG")

    # Initialize new game
    def init_game(self):
        self.player = Player(self)
        self.obstacles = []
        self.game_speed = 15
        self.points = 0
        self.x_pos_bg = 0

    # Draw background
    def draw_background(self):
        bg = self.images["CITY_BG"]
        w = bg.width()
        self.canvas.create_image(self.x_pos_bg, self.BG_Y, image=bg, anchor="nw")
        self.canvas.create_image(self.x_pos_bg + w, self.BG_Y, image=bg, anchor="nw")
        self.x_pos_bg -= self.game_speed
        if self.x_pos_bg <= -w:
            self.x_pos_bg = 0

    # Draw score
    def draw_score(self):
        self.points += 1
        if self.points > self.high_score:
            self.high_score = self.points
        if self.points % 100 == 0:
            self.game_speed += 2

        self.canvas.create_text(950, 30, text=f"Score: {self.points}", font=("Arial", 18))
        self.canvas.create_text(950, 60, text=f"High Score: {self.high_score}", font=("Arial", 18), fill="darkred")

    # Start menu screen
    def show_menu(self):
        self.canvas.delete("all")
        self.canvas.create_text(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2,
                                text="Press START to play", font=("Arial", 28, "bold"))

        self.canvas.create_image(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2 - 150,
                                 image=self.images["GIRL_RUN1"], anchor="s")

        # START button centered
        self.start_button = Button(
            self.root, text="START",
            font=("Arial", 18, "bold"),
            bg="green", fg="white",
            width=12, height=2,
            command=self.start_from_menu
        )
        self.start_button.place(relx=0.5, rely=0.6, anchor="center")

    # Game over screen
    def show_lost_screen(self):
        self.canvas.delete("all")
        self.canvas.create_text(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2,
                                text="YOU LOST", font=("Arial", 36, "bold"), fill="red")
        self.canvas.create_text(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2 + 50,
                                text="Press PLAY AGAIN", font=("Arial", 22))
        self.canvas.create_text(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2 + 100,
                                text=f"Score: {self.points}", font=("Arial", 16))
        self.canvas.create_text(self.SCREEN_WIDTH // 2, self.SCREEN_HEIGHT // 2 + 130,
                                text=f"High Score: {self.high_score}", font=("Arial", 16), fill="darkred")

        # PLAY AGAIN button centered
        self.play_again_button = Button(
            self.root, text="PLAY AGAIN",
            font=("Arial", 18, "bold"),
            bg="green", fg="white",
            width=14, height=2,
            command=self.start_after_lost
        )
        self.play_again_button.place(relx=0.5, rely=0.75, anchor="center")

    # Button functions
    def start_from_menu(self):
        self.start_button.destroy()
        self.init_game()
        self.state = "playing"
        self.game_loop()

    def start_after_lost(self):
        self.play_again_button.destroy()
        self.init_game()
        self.state = "playing"
        self.game_loop()

    # Game loop
    def game_loop(self):
        if self.state != "playing":
            return

        self.canvas.delete("all")
        self.draw_background()

        # Obstacles
        if len(self.obstacles) == 0:
            if random.randint(0, 100) < 7:
                self.obstacles.append(Obstacle(self))

        for obs in list(self.obstacles):
            obs.update()
            obs.draw()

        self.player.update()
        self.player.draw()

        # Collision
        for obs in list(self.obstacles):
            if rects_collide(self.player.get_rect(), obs.get_rect()):
                self.state = "lost"
                self.show_lost_screen()
                return

        self.draw_score()
        self.root.after(33, self.game_loop)

    # Key press
    def on_key_press(self, event):
        if self.state == "playing":
            if event.keysym in ("Up", "space", "Space"):
                self.player.start_jump()

    def on_key_release(self, event):
        pass


# Player class
class Player:
    X_POS = 150

    def __init__(self, game: Game):
        self.game = game
        self.run1 = game.images["GIRL_RUN1"]
        self.run2 = game.images["GIRL_RUN2"]
        self.jump_img = game.images["GIRL_JUMP"]

        self.Y_POS = game.GROUND_Y
        self.y = self.Y_POS
        self.x = self.X_POS

        self.JUMP_VEL_START = 22
        self.jump_vel = self.JUMP_VEL_START
        self.is_jump = False

        self.frame_counter = 0
        self.image = self.run1

    def start_jump(self):
        if not self.is_jump and self.y == self.Y_POS:
            self.is_jump = True
            self.jump_vel = self.JUMP_VEL_START

    def update(self):
        if self.is_jump:
            self.jump()
        else:
            self.run()

    def run(self):
        self.frame_counter += 1
        if self.frame_counter % 20 < 10:
            self.image = self.run1
        else:
            self.image = self.run2
        self.y = self.Y_POS

    def jump(self):
        self.image = self.jump_img
        self.y -= self.jump_vel
        self.jump_vel -= 2
        if self.y >= self.Y_POS:
            self.y = self.Y_POS
            self.is_jump = False
            self.jump_vel = self.JUMP_VEL_START

    def draw(self):
        self.game.canvas.create_image(self.x, self.y, image=self.image, anchor="s")

    def get_rect(self):
        w = self.image.width()
        h = self.image.height()
        return (self.x - w // 2, self.y - h, w, h)


# Obstacle class
class Obstacle:
    def __init__(self, game: Game):
        self.game = game
        self.image = game.images["BARRIER"]
        self.x = game.SCREEN_WIDTH + 50
        self.y = game.GROUND_Y

    def update(self):
        self.x -= self.game.game_speed
        if self.x < -self.image.width():
            if self in self.game.obstacles:
                self.game.obstacles.remove(self)

    def draw(self):
        self.game.canvas.create_image(self.x, self.y, image=self.image, anchor="s")

    def get_rect(self):
        w = self.image.width()
        h = self.image.height()
        return (self.x - w // 2, self.y - h, w, h)


# Run game
if __name__ == "__main__":
    Game()
