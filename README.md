import kivy

from kivy.app import App

from kivy.uix.widget import Widget

from kivy.clock import Clock

from kivy.graphics import Rectangle, Ellipse

from kivy.core.window import Window

import random

import datetime

 

kivy.require('1.11.1')

 

# Paramètres du jeu

width, height = 800, 600

player_width, player_height = 50, 50

obstacle_width, obstacle_height = 50, 50

coin_radius = 15

death_wall_speed = 5

obstacle_speed = 5

player_speed = 5

 

class GameWidget(Widget):

    def __init__(self, **kwargs):

        super(GameWidget, self).__init__(**kwargs)

        self.player = Rectangle(pos=(width // 4, height - player_height - 10), size=(player_width, player_height))

        self.obstacles = []

        self.coins = []

        self.death_wall = Rectangle(pos=(0, 0), size=(10, height))

        self.score = 0

 

        Clock.schedule_interval(self.update, 1.0 / 60.0)

 

    def draw_obstacle(self, pos):

        return Rectangle(pos=pos, size=(obstacle_width, obstacle_height))

 

    def draw_coin(self, pos):

        return Ellipse(pos=pos, size=(coin_radius * 2, coin_radius * 2))

 

    def draw_player(self, pos):

        return Rectangle(pos=pos, size=(player_width, player_height))

 

    def update(self, dt):

        # Génération aléatoire d'obstacles

        if random.randint(0, 100) < 5:

            obstacle_y = random.randint(0, height - obstacle_height)

            self.obstacles.append(self.draw_obstacle(pos=(width, obstacle_y)))

 

        # Génération aléatoire de pièces

        if random.randint(0, 100) < 2:

            coin_y = random.randint(0, height - coin_radius)

            self.coins.append(self.draw_coin(pos=(width, coin_y)))

 

        # Génération aléatoire d'obstacles plus fréquente pour augmenter la difficulté

        if random.randint(0, 100) < 10:

            obstacle_y = random.randint(0, height - obstacle_height)

            self.obstacles.append(self.draw_obstacle(pos=(width, obstacle_y)))

 

        # Déplacement des obstacles

        for obstacle in self.obstacles:

            obstacle.pos = (obstacle.pos[0] - obstacle_speed, obstacle.pos[1])

 

        # Déplacement des pièces

        for coin in self.coins:

            coin.pos = (coin.pos[0] - obstacle_speed, coin.pos[1])

 

        # Déplacement du mur de la mort

        self.death_wall.pos = (self.death_wall.pos[0] + death_wall_speed, 0)

 

        # Vérification des collisions avec les obstacles

        for obstacle in self.obstacles:

            if self.player.collide_widget(obstacle):

                self.reset_game()

 

        # Vérification des collisions avec les pièces

        for coin in self.coins:

            if self.player.collide_widget(coin):

                self.coins.remove(coin)

                self.score += 10

 

        # Vérification des collisions avec le mur de la mort

        if self.player.collide_widget(self.death_wall):

            self.reset_game()

 

    def reset_game(self):

        # Enregistrement du score dans un fichier texte

        with open("score.txt", "a") as file:

            file.write(f"Score: {self.score}, Date: {datetime.datetime.now()}\n")

 

        # Réinitialisation du jeu

        self.score = 0

        self.obstacles = []

        self.coins = []

        self.player.pos = (width // 4, height - player_height - 10)

        self.death_wall.pos = (0, 0)

 

    def on_touch_down(self, touch):

        if touch.x < width / 2:

            self.player.pos = (self.player.pos[0] - player_speed, self.player.pos[1])

        else:

            self.player.pos = (self.player.pos[0] + player_speed, self.player.pos[1])

 

class MyApp(App):

    def build(self):

        game = GameWidget()

        Window.size = (width, height)

        return game

 

if __name__ == '__main__':

    MyApp().run()
