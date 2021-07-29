---

layout: post
title: "Understanding the 2D Camera in Raylib"
date:   2021-07-28 22:35:00 +0000
categories: tutorial

---




This is an intro tutorial into the 2D camera in the [raylib library](https://www.raylib.com/). Raylib is a great, simple and easy to use C library for game development. The 2D camera feature itself is quite simple, however if you have never used it before it can be hard to get your head around. The aim of this post is to get you comfortable with basics of the 2d camera to get you started. All the code shown here will be on the github repo. There will be folder for each code change. When the folder I am working it changes I will let you know so you can keep in the loop. So let's get going. 



## Setting the Scene

To start, let's set the scene without the camera. Here is a start of a basic game. You have your player, a red box, and you can move it around. Up, down, left, right. If you already know this, feel free to skip to the next part.

<video style="display:block; margin: 0 auto; padding: 0px 0px 20px 0px;" controls>
  <source src="/assets/raylib-2dcamera/pre_camera.webm" type="video/mp4">
</video>

{:refdef: style="text-align: center;"}
**[1-Basic-Scene](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/1-Basic-Scene)**
{: refdef}

Let's go over the main parts of the code, so we are all on the same page. I'm going to skip some of it to save time, but you can view it all on [github](repo.com)

``` c

while(!WindowShouldClose()){

	update();

	draw();

}
```

This is the main loop. It does two things, updates and draws. The `update()` runs the show. This is where we manage inputs and our player. The `draw()` function takes this information and draws it to the screen, so we can see it. Let's break down these two functions. 

```c
void update(){
	float deltaTime = GetFrameTime();

	if (IsKeyDown(KEY_UP)) {
		player.position.y -= deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_DOWN)){
		player.position.y += deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_LEFT)) {
		player.position.x -= deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_RIGHT)){
		player.position.x += deltaTime*PLAYER_MOVE_SPEED;
	}
}
```
Super simple. First we get the delta time using the raylib's provided `GetFrameTime()` function. We won't delve too deep into this as but [there](https://dev.to/dsaghliani/understanding-delta-time-in-games-3olf) are [plenty](https://gamedev.stackexchange.com/questions/13008/how-to-get-and-use-delta-time) of [good](https://gameprogrammingpatterns.com/game-loop.html) blog posts out that explain this concept. 

But the gist is that delta time is the time of each frame and we can use that to ensure that actions happen at the same speed no matter the frame rate. 

So once we have the delta time, we check to see if any of the arrow keys are being held down and if they are, we move the player an amount multiplied by the delta time in the direction the user pressed. That's it for now.

```c
void draw(){
	BeginDrawing();

	ClearBackground(SKYBLUE);
	
	draw_border();

	DrawRectangle(player.position.x - PLAYER_SIZE/2, player.position.y - PLAYER_SIZE/2, PLAYER_SIZE, PLAYER_SIZE, RED);

	EndDrawing();
}
```

Now the draw function is just as simple. We start with the raylib function `BeginDrawing()`. This sets up the frame buffer for us to start adding things to the next frame. Next we clear the background, draw a boarder around the frame (We will get back to the border later) and draw the player. Then to finish it off, we call the function `EndDrawing()`, letting raylib know that we are done drawing, and it can swap the current frame buffer to the one we just drew on. This is called [double buffering](https://wiki.osdev.org/Double_Buffering). 

That's all there is too it really. But there is now an issue, let's say we want to move somewhere past the pink boarder. Let's see what happens. 

<video style="display:block; margin: 0 auto; padding: 0px 0px 20px 0px;" controls>
  <source src="/assets/raylib-2dcamera/great_escape.webm" type="video/mp4">
</video>


And off we go. We have now exited the screen. So what happened? Well, it's simple, as we saw on the update function. If the right arrow is the pressed, we increase the x position of the player, and eventually that position will be greater than the screen width. Therefore, it will no longer to be visible.

So how do we overcome this, the camera of course. 

## Introducing the Camera
In a 2D video game, you have two coordinate systems. You have the world and you have the camera. With a simple game like Tetris or Snake, the world and the camera are the same. But once, your level is bigger than the screen it's being displayed on. The camera coordinate system becomes a subset of the world coordinate system. The Pokémon games are a great example of this. As you move through the level, the camera follows you. You can see this in the image below. The top left-hand corner is the world origin (this isn't actually where it is in the real game, just put there for this example) and then you have the camera centered around the player, with its own origin in the top left-hand corner.

{:refdef: style="text-align: center;"}
![](/assets/raylib-2dcamera/Pokemon.png)
{: refdef}

In this example the player could be at the coordinates (2000, 1000) in the world whilst at the coordinates (500, 500) in the camera's reference.  

Okay now let's bring raylib into this. Management of the camera is done using two functions: `BeginMode2D(Camera2D camera)` and `EndMode2D(void)`. As you can see the `BeginMode2D()` function takes the structure `Camera2D`. This structure contains all the information about the camera and is made up of. We can find the definition of this structure in the `raylib.h` file.

```c
// Camera2D, defines position/orientation in 2d space
typedef struct Camera2D {
    Vector2 offset;         // Camera offset (displacement from target)
    Vector2 target;         // Camera target (rotation and zoom origin)
    float rotation;         // Camera rotation in degrees
    float zoom;             // Camera zoom (scaling), should be 1.0f by default
} Camera2D;
```

Pretty simple, we just have two `Vector2` members. One for offset and one for target. And two `float` members. One for rotation and one for zoom. We will delve into these soon. 

So let's put this into our game. First we will add a global variable for the camera itself `Camera2D camera;` to the top of the code. Next, we need to set the values we want the camera to start with. 
```c
camera.offset = (Vector2){0 , 0};
camera.target = (Vector2){0 , 0};
camera.rotation = 0.0f;
camera.zoom = 1.0f;
```
I have set everything to zero but the zoom, because it says in the camera structure definition this should be 1.0f by default (always read the docs!). This goes into my `init()` function, which is called right at the start of the program before we enter the main game loop as we only need it to be run once. 

Next, we add the two functions that raylib provides to our `draw()` function, as the camera is all about what is being drawn where. 
```c
// Draw what needs to drawn to the screen
void draw(){
	BeginDrawing();

	ClearBackground(SKYBLUE);

	BeginMode2D(camera);

	draw_border();
	 
	DrawRectangle(player.position.x - PLAYER_SIZE/2, player.position.y - PLAYER_SIZE/2, PLAYER_SIZE, PLAYER_SIZE, RED);

	EndMode2D();

	EndDrawing();
}	
```
Very similar to what we had before. We have now added the `BeginMode2D()` function after the `ClearBackground()` function. This is because we want to clear the frame right before anything else happens, so putting it before `BeginMode2D()` just makes sure that we stick to that. Then just before  `EndDrawing()` we added `EndMode2D()`. Now everything between our two new functions will be drawn with respect to the game world and not the screen. Let's see what this looks like. 

{:refdef: style="text-align: center;"}
![](/assets/raylib-2dcamera/Camera_Default.png)
{: refdef}

{:refdef: style="text-align: center;"}
**[2-Camera-Added](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/2-Camera-Added)**
{: refdef}

And it's the same... This makes sense, as all we have done is put the camera in the same position as the world origin. So now let's make it follow the player. To do so, we need to make two changes. First, to the `init()` function. We make the `camera.target` the `player.position`.

```c
camera.offset = (Vector2){0 , 0};
camera.target = player.position;
camera.rotation = 0.0f;
camera.zoom = 1.0f;
```
Next, we add the same line to the `update()` function, so it will keep it in sync as we move the player around.
```c
// Update everything that changes
void update(){
	float deltaTime = GetFrameTime();

	if (IsKeyDown(KEY_UP)) {
		player.position.y -= deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_DOWN)){
		player.position.y += deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_LEFT)) {
		player.position.x -= deltaTime*PLAYER_MOVE_SPEED;
	}

	if (IsKeyDown(KEY_RIGHT)){
		player.position.x += deltaTime*PLAYER_MOVE_SPEED;
	}

	camera.target = player.position;
}
```

<video style="display:block; margin: 0 auto; padding: 0px 0px 20px 0px;" controls>
  <source src="/assets/raylib-2dcamera/top_left_target.webm" type="video/mp4">
</video>

{:refdef: style="text-align: center;"}
**[3-Camera-Follow-Player-1](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/3-Camera-Follow-Player-1)**
{: refdef}


Well that's no idea. Things are moving, but not the things we want. So what's going on. 

Let's first take a look at the pink boarder. As I said earlier, everything inside the `BeginMode2D()` and `EndMode2D()` functions is drawn with respect to the world origin and not the screen like it use to. So the boarder is staying in its place which is at the world origin. To fix this and draw it relative to the screen again, all we need to do is move it, so it's no longer between `BeginMode2D()` and `EndMode2D()`. So here is the updated code. 

```c
// Draw what needs to drawn to the screen
void draw(){
	BeginDrawing();

	ClearBackground(SKYBLUE);

	draw_border();

	BeginMode2D(camera);
	 
	DrawRectangle(player.position.x - PLAYER_SIZE/2, player.position.y - PLAYER_SIZE/2, PLAYER_SIZE, PLAYER_SIZE, RED);

	EndMode2D();

	EndDrawing();
}	
```

Let's give it a spin.

{:refdef: style="text-align: center;"}
![](/assets/raylib-2dcamera/Boarders_back.png)
{: refdef}

{:refdef: style="text-align: center;"}
**[3-Camera-Follow-Player-2](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/3-Camera-Follow-Player-2)**
{: refdef}

That's a better. Now let's fix the player. So what's happening is the camera's origin and the player's position are the same. Because the origin of the camera is in the top left-hand corner, that is also where the player is. We fix this with the `offset` member of our camera structure. This lets us move where the origin of the camera is relative to our screen. So instead of the top left-hand corner, we can move to the bottom left-hand corner or the coordinates {458, 186} or what might be more useful for us is the center. To do so, we just need to modify one line of code, which is the camera initialization code in our `init()` function to this:
```c
camera.offset = (Vector2){SCREEN_WIDTH/2 ,SCREEN_HEIGHT/2};
camera.target = player.position;
camera.rotation = 0.0f;
camera.zoom = 1.0f;
```

<video style="display:block; margin: 0 auto; padding: 0px 0px 20px 0px;" controls>
  <source src="/assets/raylib-2dcamera/player_center_following.webm" type="video/mp4">
</video>

{:refdef: style="text-align: center;"}
**[3-Camera-Follow-Player-3](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/3-Camera-Follow-Player-3)**
{: refdef}

And there we have it, we are following the player. If you are wondering where those circles came from, I add a function to generate and draw random circles in the world, so we could see the movement more clearly (You can checkout the [github](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/3-Camera-Follow-Player-3) for the code used.).

## Closing 

So we have gone from the player moving off the side of the screen, to the camera following the player is less the than 10 lines of code. Nice!

So what did we learn (I hope):
- The camera has its own coordinate system that is a subset of the world's system.
- Everything in-between the `BeginMode2D()` and `EndMode2D()` is draw relative to the world coordinates.
- The `target` member of the `Camera2D` structure sets where the origin of the camera is in the world coordinate system.
- The `offset` member of the `Camera2D` structure sets where the origin of the camera is on the screen. 

There is a lot more you can do with the 2D camera and I would suggest you play around with the provided code to see what you can come up with. If you would like some inspiration, this Google doc called [[Scroll Back - The Theory and Practice of Cameras in Side-Scrollers by Itay Keren]] is a great showcase of different camera usage in side scrollers. I have also added a [BONUS](https://github.com/ArchieAtkinson/raylib-2dcamera-tutorial/tree/main/BONUS) folder to the github which has an extra demo for you to test out. For some more complex examples, check out the 2d camera_platform example on the raylib git or [on the website](https://www.raylib.com/examples.html). 

I hope you found this helpful, if you have any questions, comments or feedback feel free to [email](mailto:archieatkinson@gmail.com) or leave a comment on the [blogs github issue tracker](https://github.com/ArchieAtkinson/archieatkinson.github.io/issues/2)(I will work out proper commenting system one day...).


P.S An extra trick is the `GetWorldToScreen2D()` and the `GetScreenToWorld2D()` functions to help you move between the two coordinate systems. 