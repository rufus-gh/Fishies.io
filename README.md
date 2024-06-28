# Fishies.io Development Logbook

## May 1, 2024
Decided to create a game called `fishies.io` in Unity2D. The concept is simple: you start as a small fish and grow by eating smaller enemies while avoiding bigger ones. The player controls the fish with the mouse cursor, and when you eat a fish bigger than you, your size increases.

## May 5, 2024
Started working on the basic mechanics of the game. Implemented the movement of the fish following the mouse cursor using Unity's `Input.mousePosition` property and smoothed it using linear interpolation

## May 10, 2024
Added enemy fish that swim around randomly. They pick a random direction and go towards it at a random speed within a range. I also made the camera smoothly follow the player using smooth linear interpolation, and made it zoom out as the player's size increase in order to keep them in the frame.

## May 15, 2024
Created an enemy spawning system that works by creating a donut like shape around the player, so that the enemy doesn't spawn too close to the player but also doesn't spawn too far away. The enemies also get automatically killed if they go outside of the bounds, which helps incase they swim outside the area or spawn outside the game area. 

## May 20, 2024
Implemented the eating mechanic. If the player's fish is bigger than the enemy, the enemy gets eaten and the player grows. If the enemy is bigger, the player gets eaten and the game ends.

## May 25, 2024
Decided to add a new mechanic: holding the left mouse button makes the player's fish boost forwards but decreases the score. This adds a risk-reward element to the game, as you can boost towards smaller fish and try to eat them, but if you boost for too long you may become smaller than them. The fish also spawn with a random size ranging from 0.5 to 1.5 times the player's current size, to add food as well as threats to keep it interesting.

## May 30, 2024
Implemented the new mechanic. Used Unity's `Input.GetMouseButton` function to check if the left mouse button is being held down, and decreased their score over time while making them move twice as fast.

## June 5, 2024
Made some changes to the scoring system. Eating a bigger enemy now gives more score than eating a smaller one. This encourages the player to take risks, and keeps the player's score steadily increasing as they grow bigger.

## June 10, 2024
Started working on the game's UI. Added a score display and a main menu screen which the player returns to when they die.

## June 15, 2024
Made the player's speed increase as they grow bigger so that they don't seem to slow down compared to their size. I also removed the smooth camera follow as it wasn't behaving properly and would become jerky whenever the player's size and speed increased.

## June 20, 2024
Did some more playtesting, and discovered that it was very easy to become the size of the arena, which would cause you to collide with the walls and die, so I expanded the arena size. Due to the much larger arena, I added some optimization by turning off the renderer of the enemies when they are out of the view distance of the player to reduce the calculations needed to check if they need to be drawn to the screen.

## June 25, 2024
Added graphics for the player as well as the enemies, and improved the start menu ui.

## June 28, 2024
Did a bit of playtesting and found that the game is now quite well balanced and enjoyable. In the future I plan on adding sound effects, an online leaderboard (to add a more competitive aspect to the game), and a shop where you upgrade your speed and starting size, as well as buy powerups and cosmetics. Coins could be obtained through eating enemies, and maybe boss battles with more exotic fish to grant bonus rewards.

# References
- Unity Documentation: Input.mousePosition
- Unity Documentation: Input.GetMouseButton
