# Fishies.io Development Logbook

I created this game using Unity Engine 2D, which is a game engine that you program with C#. I learnt a lot about both how Unity works as well as how to use C# and it's syntax throughout the development of my game.

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

# Evaluation
Overall I think this is a very good game and I had a lot of fun making it. I also learnt a lot along the way about how to design, develop, manage, and evaluate software. I evaluated my game by having my family members playtest the game, and they got the hang of it after a few tries and ended up quite enjoying it.

# Game Scripts

## Player Script (MouseMovement.cs)
This is the script that controls everything about the player, such as movement, boosting, growing size, eating other fish, and dying.
```cs
// import the modules required for unity
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
// import the module required to switch back to the main menu screen which happens when the player dies
using UnityEngine.SceneManagement;

// create a mouse movement class for the player
public class MouseMovement : MonoBehaviour
{
    // set the speed of the player
    private float speed = 5f;
    // set the initial speed of the player
    public float initialSpeed = 5f;
    // set the rigidbody module
    private Rigidbody2D rb;
    // set the distance limit for the mouse so that it doesn't glitch out when the mouse is too close to the player
    public float distanceLimit = 1f;
    // set the size of the player
    public float size = 1f;
    // set a boolean to check if the player is being boosted
    private bool boosted;

    // this runs once at the start
    void Start()
    {
        // get the rigidbody module
        rb = GetComponent<Rigidbody2D>();
    }

    // function to run when the player dies
    void Die() {
        // switch back to the main menu screen when the player dies
        SceneManager.LoadScene("Main Menu");
    }

    // when the player collides with a trigger
    void OnTriggerEnter2D(Collider2D col)
    {
        // check if it is the wall
        if (col.gameObject.tag == "Wall")
        {
            // if so then kill the player
            Die();
        }
    }

    // when the player collides with an object
    void OnCollisionEnter2D(Collision2D col)
    {
        // check if it is an enemy
        if (col.gameObject.tag == "Enemy")
        {
            // if the player is bigger than the enemy
            if (transform.localScale.x > col.gameObject.transform.localScale.y) {
                // kill the enemy and increase the player's size
                size += 0.1f*Mathf.Round(col.gameObject.transform.localScale.y);
                Destroy(col.gameObject);
            } else {
                // otherwise if the player is smaller then die
                Die();
            }
        }
    }

    // runs every frame of the game
    void Update()
    {
        // set the size of the player based on the player variable
        transform.localScale = new Vector3(size, size, 1f);
        // lerp the speed for smooth transitions
        speed = Mathf.Lerp(speed, initialSpeed*size, Time.deltaTime);

        // check if they are holding left click, if they are then boost them, if not then don't boost them
        if (Input.GetMouseButton(0)) {
            boosted = true;
        } else {
            boosted = false;
        }

        // Get the Screen positions of the object and the mouse
        Vector3 playerScreenPos = Camera.main.WorldToScreenPoint(transform.position);

        // check if the players mouse is within the limit, if so then don't move it, otherwise, move it towards the mouse
        if (Vector2.Distance(Input.mousePosition, playerScreenPos) < distanceLimit) {
            // stop the player from moving
            rb.velocity = new Vector3(0f, 0f, 0f);
        } else {
            // find the direction vector for which the player needs to move in
            Vector3 direction = Input.mousePosition - playerScreenPos;

            // Calculate the angle of the rotation
            float angle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;

            // Apply the rotation to the player
            Quaternion rotation = Quaternion.AngleAxis(angle, Vector3.forward);
            transform.rotation = Quaternion.Slerp(transform.rotation, rotation, speed * Time.deltaTime);

            // Move the player towards the mouse
            Vector2 moveDirection = new Vector2(direction.x, direction.y).normalized;

            // move the player forwards, if it is boosting then move it twice as fast
            if (boosted) {
                rb.MovePosition(rb.position + moveDirection * speed * Time.deltaTime * 2f);
                // slowly reduce their size as they boost (similar to the boost mechanic in slither.io)
                size -= size * 0.05f * Time.deltaTime;
            } else {
                rb.MovePosition(rb.position + moveDirection * speed * Time.deltaTime);
            }
        }
 
    }

}

```

## Camera Follow Script (FollowPlayer.cs)

This script makes the camera follow the player.

```cs
// import required libraries for Unity
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// create a class that will be attached to the camera to make it follow the player
public class FollowPlayer : MonoBehaviour
{
    // get the player gameobject
    public GameObject player;
    // prepare to store the mousemovement script
    private MouseMovement mm;
    // set a changeable startZoom amount to control how much the zoom starts out as
    public float startZoom;
    // create a variable that changes the zoom scaling based on the players size
    public float zoomAmount;
    // get the camera module of the gameobject
    private Camera cam;

    // Start is called before the first frame update
    void Start()
    {
        // get the mousemovement script
        mm = player.GetComponent<MouseMovement>();
        // get the camera module
        cam = GetComponent<Camera>();
    }

    // Update is called once per frame
    void Update()
    {
        // these were my initial scripts to make the camera follow the player however as the player gained size and
        // sped up the camera would not follow behind the player properly
        //transform.position = Vector2.Lerp(transform.position, player.transform.position, Time.deltaTime/mm.size);
        //transform.position = new Vector3(transform.position.x, transform.position.y, -10f);

        // this is the final script that I used to make the camera follow the player
        // set the camera's x and y coordinates to the player's x and y coordinates
        transform.position = new Vector3(player.transform.position.x, player.transform.position.y, -10f);
        // change the camera zoom based on the player's size and the zoom amount variable
        cam.orthographicSize = startZoom + mm.size*zoomAmount;
    }
}
```

## Optimization Derendering (OffscreenDeload.cs)

This is the script that deloads the enemies if they are too far away from the player

```cs
// import the modules needed for unity
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// setup the class that will deload enemies if they are too far away from the player in order to increase performance
public class OffscreenDeload : MonoBehaviour
{
    // get the gameobject of the player in order to get their position
    GameObject player;
    // get the enemies sprite renderer in order to disable it if the player is far enough away
    SpriteRenderer spriteRenderer;

    // runs on program start
    void Start()
    {
        // find the player gameobject using an nth level tree search
        player = GameObject.Find("Player");
        // find the sprite renderer of the player
        spriteRenderer = gameObject.GetComponent<SpriteRenderer>();
    }

    void Update()
    {
        // check the distance to the player, and if it is greater than 21, hide the player
        if (Vector2.Distance(player.transform.position, transform.position) > 21*player.size) {
            spriteRenderer.enabled = false;
        } else {
            spriteRenderer.enabled = true;
        }
    }

}
```

## Enemy Spawner (RandomSpawner.cs)

This is the script that spawns the enemies.

```cs
// import required module for unity
using UnityEngine;

// this class randomly spawns enemies
public class RandomObjectSpawner : MonoBehaviour
{
    public GameObject prefab; // The prefab to instantiate
    public Vector2 radiusRange; // The range for x
    public Vector2 sizeRange; // The range for the size
    public GameObject player; // The player gameobject
    private MouseMovement mm; // The mousemovement script

    void Start()
    {
        // get the mouse movement script of the player to get it's position and size
        mm = player.GetComponent<MouseMovement>();
        
        // spawn 50 enemies on the map at the start of the game
        for (int i=0;i<50;i++) {
            SpawnObject();
        }

        // Call the SpawnObject method every second
        InvokeRepeating("SpawnObject", 0f, 5f);
    }

    // function to spawn enemies
    void SpawnObject()
    {
        // Generate a random position within the specified ranges
        float angle = Random.Range(0, 360);
        float radius = Random.Range(radiusRange.x, radiusRange.y);
        Vector2 position = new Vector2(radius*Mathf.Cos(Mathf.Rad2Deg*angle)+player.transform.position.x, radius*Mathf.Sin(Mathf.Rad2Deg*angle)+player.transform.position.y);

        // Instantiate the prefab at the random position
        GameObject instance = Instantiate(prefab, position, Quaternion.identity);

        // Generate a random size within the specified range
        float size = Random.Range(sizeRange.x, sizeRange.y*mm.size);

        // Set the size of the instance
        instance.transform.localScale = new Vector3(size, size, size);
    }
}
```

## Scene Switching Script (SceneSwitcher.cs)

This is the script that runs when the user presses the start button

```cs
using UnityEngine;
using UnityEngine.SceneManagement;

// script that runs when the start button is clicked to switch to the game scene
public class SceneSwitcher : MonoBehaviour
{
    public void SwitchScene()
    {
        // Load the "Game" scene
        SceneManager.LoadScene("Game");
    }
}
```

## Score Display (ShowScore.cs)

This is the script that shows the players score on the screen

```cs
// importing necessary libraries
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

// defining a public class showscore which inherits from monobehaviour
public class ShowScore : MonoBehaviour
{
    // declaring a public gameobject variable 'player'
    public GameObject player;
    // declaring a private mousemovement variable 'mm'
    private MouseMovement mm;
    // declaring a public textmeshprougui variable 'mtext'
    public TextMeshProUGUI mText;

    // start is a method called before the first frame update
    void Start()
    {
        // getting the mousemovement component attached to the 'player' gameobject
        // and assigning it to the 'mm' variable
        mm = player.GetComponent<MouseMovement>();
    }

    // update is a method called once per frame
    void Update()
    {
        // updating the text of 'mtext' to display the score
        // the score is calculated by rounding the 'size' property of 'mm' multiplied by 10
        // the result is then converted to a string
        mText.text = "Score: "+(Mathf.Round(mm.size*10f)).ToString();
    }
}
```

## Enemy Movement Script (Wander.cs)

This is the script that makes the enemies wander randomly around the screen as well as destroys them if they are outside of the bounds.

```cs
// importing necessary libraries
using System.Collections;
using UnityEngine;

// defining a public class wander which inherits from monobehaviour
public class Wander : MonoBehaviour
{
    // declaring a public float variable 'speed' and initializing it to 5.0f
    public float speed = 5.0f;
    // declaring a public float variable 'changeDirectionInterval' and initializing it to 1.0f
    public float changeDirectionInterval = 1.0f;

    // declaring a private vector2 variable 'direction'
    private Vector2 direction;
    // declaring a private vector2 variable 'smoothDirection'
    private Vector2 smoothDirection;

    // start is a method called before the first frame update
    void Start()
    {
        // starting the coroutine 'changedirection'
        StartCoroutine(ChangeDirection());
    }

    // update is a method called once per frame
    void Update()
    {
        // updating the 'smoothdirection' by interpolating between its current value and 'direction'
        smoothDirection = Vector2.Lerp(smoothDirection, direction, Time.deltaTime);
        // translating the transform in the direction of 'smoothdirection' at a speed of 'speed' units per second
        transform.Translate(smoothDirection * speed * Time.deltaTime);

        // checking if the transform's position is outside a certain range
        // if it is, the gameobject is destroyed
        if (transform.position.x > 40 || transform.position.x < -40 || transform.position.y > 40 || transform.position.y < 40) {
            Destroy(gameObject);
        }
    }

    // changedirection is a coroutine that changes the 'direction' at an interval specified by 'changedirectioninterval'
    IEnumerator ChangeDirection()
    {
        // infinite loop
        while (true)
        {
            // setting 'direction' to a new random vector2
            direction = new Vector2(Random.Range(-1f, 1f), Random.Range(-1f, 1f));
            // waiting for 'changedirectioninterval' seconds before the next iteration
            yield return new WaitForSeconds(changeDirectionInterval);
        }
    }
}
```

# References
- Unity Documentation: Input.mousePosition
- Unity Documentation: Input.GetMouseButton
- https://blog.logrocket.com/detect-mouse-movement-input-unity/
- https://docs.unity3d.com/2021.1/Documentation/Manual/Quickstart2DCreate.html
- https://learn.unity.com/tutorial/working-with-textmesh-pro
- https://forum.unity.com/threads/best-way-to-get-variable-from-another-script-c-unity.579322/
- https://www.pinterest.com.au/pin/586030970236850636/
- https://discussions.unity.com/t/how-to-switch-between-scenes/189643
- https://medium.com/@michael.desena.dev/creating-ui-in-unity-6a51992db40f
