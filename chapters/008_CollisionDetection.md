\null\clearpage

Collision Detection and Reaction
=================================

\epigraph{Every detection of what is false directs us towards what is true: every trial exhausts some tempting form of error.}{\textit{William Whewell}}

When it comes to collision management, there are two main phases:

- **Collision Detection:** you find out which game objects collided with each other;
- **Collision Reaction:** you handle the physics behind the collision detected, making the game objects react to such collision.

Collisions don't only happen between game objects (two fighters hitting each other), but also between a character and the world (or they would end up just going through the ground).

In this section we'll talk about some ways you can detect and react to collisions.

Collision Detection: Did it really collide?
--------------------------------------------

First of all, we need to see how we can make sure that two objects really collide with each other.

### Collision Between Two Points

This is the simplest case: points are mono-dimensional objects, and the only way two points can collide is when they have the same coordinates.

An example algorithm would be the following:

\code{collisiondetection/point_to_point}{Point to point collision detection}

A possible lazy/shorter version could be:

\code{collisiondetection/point_to_point_lazy}{Shortened version of a point to point collision detection}

This algorithm consists in a constant number of operations, so it runs in O(1).

Since numbers in computers can be **really** precise, a collision between two points may be a bit too precise, so it could prove useful to have a "buffer" around the point, so that we can say that the two points collided when they're **around the same place**.

<!-- TODO: Add point/point collision with buffer zone -->

### Collision Between A Point and a Circle

Now a circle comes into the mix, a circle has two major characteristics: a **center** and a **radius**.

> A point is considered inside of a circle when the distance between the point and the center of the circle is *less than or equal* to the radius.

So we need a function that calculates the distance between two points, and then use it to define if a point is inside a circle.

An example could be the following:

\code{collisiondetection/point_circle}{Point to circle collision detection}

Again, the lazier version:

\code{collisiondetection/point_circle_lazy}{Shorter version of a point to circle collision detection}

Although slightly more heavy, computation-wise, this algorithm still runs in O(1).

### Collision Between Two Circles

Let's add another circle into the mix now, we can declare:

> Two circles are colliding when the distance between their centers is less or equal the sum of their radii

In pseudo code this would be:

\code{collisiondetection/circle_circle}{Circle to Circle Collision Detection}

The shorter version would be:

\code{collisiondetection/circle_circle_lazy}{Shorter Version of a Circle to Circle Collision Detection}

Again, this algorithm performs a number of operations that is constant, so it runs in O(1).


### Collision Between Two Axis-Aligned Rectangles (AABB)

This is one of the most used types of collision detection used in games: it's a bit more involved than other types of collision detection, but it's still computationally easy to perform. This is usually called the "Axis Aligned Bounding Box" collision detection, or AABB.

Let's start with a bit of theory. We have two squares:

![Example used in the AABB collision detection](./images/collision_detection/AABB1.pdf){width=30%}

To know if we may have a collision, we need to check if one of the sides is "inside" (that means between the top and bottom sides) of another rectangle:

![Top-Bottom Check](./images/collision_detection/AABB2.pdf){width=30%}

In this case we know that the "top side" of the second rectangle (highlighted in blue) has a `y` coordinate between the first rectangle's top and bottom sides' `y` coordinates (highlighted in red).

Though this is a necessary condition, this is not sufficient, since we may have a situation where this condition is satisfied, but the rectangles don't collide:

![Top-Bottom Check is not enough](./images/collision_detection/AABB3.pdf){width=30%}

So we need to check the other sides also, in a similar fashion:

![An example of a left-right check](./images/collision_detection/AABB4.pdf){width=30%}

This has to happen for all four sides of one of the rectangle.

Now we can try putting down a bit of code, we'll assume that rectangles are defined by their top-left corner (as usually happens) and their width and height:

\code{collisiondetection/AABB}{Axis-Aligned Bounding Box Collision Detection}

This complex conditional checks 4 things:

- The left side of rectangle A is **at the left** of the right side of rectangle B;
- The right side of rectangle A is **at the right** of the left side of rectangle B;
- The top side of rectangle A is **over** the bottom side of rectangle B;
- The bottom side of rectangle A is **underneath** the top side of rectangle B.

If all four checks are true, then a collision happened.

The best way to understand this algorithm properly is to test it by hand and convince yourself that it works.

This is a very light algorithm but can quickly become heavy on the CPU when there are many objects to check for collision. We'll see later how to limit the number of checks and make collision detection an operation that is not as heavy on our precious CPU cycles.

### Line/Point Collision

We can represent a segment by using its two extreme points, which proves to be a quite inexpensive way to represent a line (it's just two points). Now how do we know if a point is colliding with a line?

To know if a point is colliding with a line we need... Triangles!

Every triangle can be represented with 3 points, and there is a really useful theorem that we can make use of:

> The sum of the lengths of any two sides must be greater than, or equal, to the length of the remaining side.

So, given a triangle ABC:

![Example of the triangle inequality theorem](./images/collision_detection/triangle_ineq1.pdf){width=30%}

We get the following 3 inequalities:

$$ \overline{AB} + \overline{BC} \leq \overline{AC} $$
$$ \overline{AC} + \overline{BC} \leq \overline{AB} $$
$$ \overline{AB} + \overline{AC} \leq \overline{BC} $$

What is more interesting to us is that when the one of the vertices of the triangle is **on** its opposite side, the triangle degenerates:

![Example of a degenerate triangle](./images/collision_detection/triangle_ineq2.pdf){width=30%}

And the theorem degenerates too, to the following:

$$ \overline{AC} + \overline{BC} = \overline{AB}$$

So we can calculate the distance between the point and each of the two extremes of the line and we know that when the sum of such distances is equal to the length of the line, the point will be colliding with the line.

In code, it would look something like the following:

\code{collisiondetection/line_point}{Line to Point Collision detection}

It could prove useful to put a "buffer zone" in here too, so that the collision detection doesn't result too jerky and precise.

<!-- TODO: Line vs Point with buffer zone -->

### Line/Circle Collision

As in the previous paragraph, we memorize a line as a pair of Points, so checking if the circle collides with either end of the line is easy, using the Point/Circle collision algorithm.

\code{collisiondetection/line_circle_partial}{Partial Implementation of a Line to Circle Collision Detection}

Now our next objective is finding the closest point **on the line** to the center of our circle. The details and demonstrations on the math behind this will be spared, just know the following:

Given a line $\overline{AB}$ between points $A=(x_1,y_1)$ and $B=(x_2,y_2)$ and a point $P=(x_k, y_k)$, the point on the line closest to P has coordinates:

$$x=x_1 + u \cdot (x_2 - x_1)$$
$$y=y_1 + u \cdot (y_2 - y_1)$$

With:

$$ u = \frac{(x_k - x_1)\cdot(x_2 - x_1) + (y_k-y_1)\cdot(y_2-y_1)}{||B-A||^2}$$

That's a lot of math!

We need to be careful though, cause this formula gives us the point for an *infinite* line, so the point we find could be outside of our line. We will use the line/point algorithm to check for that.

After we made sure the point is on the line, we can measure the distance between such point and the center of our circle, if such distance is less than the radius, we have a hit! (Or just apply the circle/point collision algorithm again).

The final algorithm should look something like this:

\code{collisiondetection/line_circle}{Line to circle collision detection}

### Point/Rectangle Collision

If we want to see if a point collides with a rectangle is really easy, we just need to check if the point's coordinates are inside the rectangle.

\code{collisiondetection/point_rectangle}{Point/Rectangle collision detection}

### Point/Triangle Collision

A possible way to define if a point is inside a triangle, we can use a bit of geometry.

We can use *Heron's formula* to calculate the area of the original triangle, and compare it with the sum of the areas created by the 3 triangles made from 2 points of the original triangle and the point we are testing.

![Point/Triangle Collision Detection: division into sub-triangles](./images/collision_detection/point_triangle.png){width=30%}

If the sum of the 3 areas (represented in different colors in the figure) equals to the original calculated area, then we know that the point is inside the triangle.

Let's see the code:

\code{collisiondetection/point_triangle}{Point/Triangle Collision Detection}

### Circle/Rectangle Collision

First of all we need to identify which side of the rectangle we should test against, so if the centre of the circle is to the right of the rectangle, we will test against the right edge of the rectangle, if it's above we'll test against the top edge and so on...

After that, we just perform some math on the distances and calculated values to detect if the circle collides with the rectangle.

\code{collisiondetection/rectangle_circle}{Rectangle to Circle Collision Detection}

### Line/Line Collision

Line/Line collision is quite simple to implement once you know the inner workings of geometry, but first we need to explain the thought behind this algorithm, so... **math warning!!**

Let's look at the following image:

![Example image for line/line collision](./images/collision_detection/line_line.png){width=60%}

A generic point $P_a$ of line A can be represented with the following formula:

$$ P_a = P_1 + u_a \cdot (P_2 - P_1) $$

which translates into the coordinate-based equations:

$$
\begin{cases}
x_a = x_1 + u_a \cdot (x_2 - x_1)\\
y_a = y_1 + u_a \cdot (x_2 - x_1)
\end{cases}
$$

This makes us understand that any point of line A can be represented by its starting point $P_1$, plus a certain fraction (represented by $u_a$) of the vector represented by $P_2 - P_1$.

This also means that $0 \leq u_a \leq 1$, else the point won't be on the segment.

In the same way, a generic point $P_b$ of line B can be represented with:

$$ P_b = P_3 + u_b \cdot (P_4 - P_3) $$

which becomes:

$$
\begin{cases}
x_b = x_3 + u_b \cdot (x_4 - x_3)\\
y_b = y_3 + u_b \cdot (x_4 - x_3)
\end{cases}
$$

The two lines will collide when $P_a = P_b$, so we get the following equations:

$$
\begin{cases}
x_1 + u_a \cdot (x_2 - x_1) = x_3 + u_b \cdot (x_4 - x_3)\\
y_1 + u_a \cdot (y_2 - y_1) = y_3 + u_b \cdot (y_4 - y_3)
\end{cases}
$$

That need to be solved in the $u_a$ and $u_b$ variables.

The result is:

$$
\begin{cases}
u_a = \frac{(x_4 - x_3) \cdot (y_1 - y_3) - (y_4 - y_3) \cdot (x_1 - x_3)}{(y_4 - y_3) \cdot (x_2 - x_1) - (x_4 - x_3) \cdot (y_2 - y_1)}\\
u_b = \frac{(x_2 - x_1) \cdot (y_1 - y_3) - (y_2 - y_1) \cdot (x_1 - x_3)}{(y_4 - y_3) \cdot (x_2 - x_1) - (x_4 - x_3) \cdot (y_2 - y_1)}
\end{cases}
$$

Substituting either of the results in the corresponding equation for the line will give us the intersection point (which may be useful for some particle effects).

Now some notes on our solution:

- If the denominator for the equations for $u_a$ and $u_b$ equals to zero, the two lines are parallel
- If both the numerator and denominator for $u_a$ and $u_b$ are equal to zero, the two lines are coincident
- If both $0 \leq u_a \leq 1$ and $0 \leq u_b \leq 1$ then the two segments collide.

Now we can translate all this math into code:

\code{collisiondetection/line_line}{Implementation of the line/line collision detection}

This collision detection algorithm can be useful for line-based puzzle games, line the untangle puzzle.

### Line/Rectangle Collision

Given the previous explanation about the Line/Line collision detection, it's quite easy to build a Line/Rectangle algorithm; distinguishing the cases where we want to account for a segment being completely inside of a rectangle or not.

\code{collisiondetection/line_rectangle}{Implementation of the line/rectangle collision detection}

This can prove useful to test for "line of sight" inside an AI algorithm.

### Point/Polygon Collision

\placeholder

### Circle/Polygon Collision

\placeholder

### Rectangle/Polygon Collision

\placeholder

### Line/Polygon Collision

\placeholder

### Polygon/Polygon Collision

\placeholder

<!-- TODO: add an algorithm to check for convex polygons colliding each other (hulls) -->

### Pixel-Perfect collision

Pixel perfect collision is the most precise type of collision detection, but it's also by far the slowest.

The usual way to perform collision detection is using **bitmasks** which are 1-bit per pixel representation of the sprites (white is usually considered a "1" while black is considered a "0").

A logic "AND" operation is performed, pixel-by-pixel, on the bitmasks; with the sprite position taken in consideration, as soon as the first AND returns a "True" a collision occurred.

\code{collisiondetection/pixel_perfect}{Example of a possibile implementation of pixel perfect collision detection}

This algorithm has a time complexity of $O(n \cdot m)$ where $n$ is the total number of pixels of the first bitmask, while $m$ is the total number of pixels in the second bitmask.

### Multi-pass collision detection

It's possible to have a more precise collision detection at a lower cost by combining different collision detection algorithms.

The most common way to apply a multi-pass collision detection is by dividing the process in a "broad" and a "fine" pass.

The broad pass can use a very simple algorithm to check for the possibility of a collision, the algorithms used are usually computationally cheap, such as:

- Circle/Circle Collision Detection
- AABB Collision detection

When the simpler algorithm detects the possibility of a collision, a more precise algorithm is used to check if a collision really happened, usually such finer algorithms are computationally expensive and will benefit from the first "broad pass" filter, thus avoiding useless heavy calculations.


Finding out who hit what
------------------------

Now we need to find which game objects collided, and this can be easily one of the most expensive parts of our game, if not handled correctly.

This section will show how knowing which items will surely **not** collide can help us optimize our algorithms.

We need to remember that each object (as good practices suggest) know only about themselves, they don't have "eyes" like us, that can see when another object is approaching them and thinking "I'm gonna collide". The only thing we can do it having "someone else" take care of checking for collisions.

As an example, we'll take the following situation:

![Example for collision detection](./images/collision_detection/collision_example.pdf){width=30%}

We can evidently see how circles 1 and 2 are colliding, but obviously our game won't just "know" without giving it a way to think about how two objects collide.

### The Brute Force Method

The simplest method is the so-called "brute force" method: you don't know which items may collide? Just try them all.

So if we consider a list of 7 game objects, we'll need to see if 1 collides with 2, 1 collides with 3, ..., 2 collides with 1, ...

An algorithm of this type could be the following:

\code{collisiondetection/brute_force}{Brute Force Method of collision search}

This algorithms runs in O(n^2^), because it checks every item with every other, even with itself.

In this example, the algorithm completes in 49 steps, but you can imagine how a game could slow down when there is an entire world to update (remember the collision detection, among with other updates and rendering/drawing, must happen in less than 16.67 and 33.33ms, so if you can save time, you totally should).

### Building Quad Trees

A nice idea would be being able to limit the number of tests we perform, since the brute force method can get really expensive really quickly.

When building quad-trees, we are essentially dividing the screen in "quadrants" (and if necessary, such quadrants will be divided into sub-quadrants), detect which objects are in such quadrants and test collisions between objects that are inside of the same quadrant.

![Graphical example of a quad tree, overlaid on the reference image](./images/collision_detection/collision_quad_example.pdf){width=40%}

With the original brute force method, we will make at most 49 tests for 7 items (although it can be optimized), while with quad trees we will perform:

- 2 tests on quadrant A (1 against 2, and 2 against 1);
- No tests on quadrant B (an object alone cannot collide against other objects);
- 2 tests on quadrant C (4 against 5, and 5 against 4);
- 6 tests on quadrant D (5-6, 5-7, 6-7, 6-5, 7-6, 7-5)

For a total of 10 tests, which can be further optimized (by 50%) by avoiding testing objects that have already been tested.

\placeholder

<!-- TODO: Easier on the CPU but harder to implement, every frame you build a quad tree
and use that to check on collisions -->

### Calculating the position of tiles

When you are using tiles to build a level, it can prove hard to be able to use quad trees or brute force methods to limit the number of collision checks inside your game.

Using a bit of math is probably the easiest and most efficient method to find out which collisions happened.

Let's take an example level:

![Example tile-based level](images/collision_detection/Tile_Calc_Example_Level_1.png){width=40%}

If a game entity is falling, like in the following example:

![Tile-based example: falling](images/collision_detection/Tile_Calc_Falling.png){width=40%}

Using the simple AABB collision detection, we will need to check only if the two lowest points of the sprite have collided with any tile in the level.

First of all let's consider a level as a 2-dimensional array of tiles and all the tiles have the same size, it is evident that we have two game entities that work with different measures: the character moves pixel-by-pixel, the ground instead uses tiles. We need something to make a conversion.

Assuming `TILE_WIDTH` and `TILE_HEIGHT` as the sizes of the single tiles, we'll have the following function:

\code{collisiondetection/tile_conversion}{Converting player coordinates into tile coordinates}

To know which tiles we need to check for collision, we just have to check the two red points (see the previous image), use the conversion function and then do a simple AABB check on them.

\code{collisiondetection/tile_collision}{Tile-based collision detection}

Considering that this algorithm calculates its own colliding tiles, we can state that its complexity is `O(n)` with `n` equal to the number of possibly colliding tiles calculated.

If an object is bigger than a single tile, like the following example:

![Example tile-based level with a bigger object](images/collision_detection/Tile_Calc_Example_Level_2.png){width=40%}

We will need to calculate a series of intermediate points (using the `TILE_WIDTH` and `TILE_HEIGHT` measures) that will be used for the test

![Tile-based example with a bigger object: falling](images/collision_detection/Tile_Calc_Falling_2.png){width=40%}

And using the same method the colliding tiles can be found without much more calculations than the previous algorithm, actually we can use exactly the same algorithm with a different list of points to test.

Collision Reaction/Correction
------------------------------

When you are sure, via any algorithm, that a collision has occurred, you now have to decide how to react to such collision. You may want to destroy the player or the target, or you may want to correct the behaviour, thus avoiding items getting inside walls.

### HitBoxes vs HurtBoxes

First of all, we need to explain the difference between a "HurtBox" and a "HitBox".

Such difference can be more or less important, depending on the game that is coded, and sometimes the two concepts can be confused.

A **HitBox** is a shape (usually a rectangle, see [Collision Between Two Axis-Aligned Rectangles (AABB)]) that is used to identify where a certain entity can *hit* another entity. For the player a "hitbox" could encase their sword while attacking.

A **HurtBox** is instead a shape that is used to identify where a certain entity can *get hurt* by another entity. For the player a "hurtbox" could be their body.

![Example of a hitbox (red) and a hurtbox (blue)](./images/collision_detection/hithurt.png){width=40%}

### Collision Reaction Methods

It has happened: a collision occurred and now the two objects are overlapping.

How do we react to this event in a convincing (not necessarily "realistic") and efficient manner? There are a lot of methods to react to collisions and below we will show some of the most used, along with some interesting ones.

We will use the following image as reference for each collision reaction:

![Images used as a reference for collision reaction](./images/collision_detection/reaction_reference.png){width=40%}

We will study each case separately, and each case will be a piece of this reference image.

#### The Direction + Velocity Method

This is the simplest method, computationally speaking: as soon as the objects gets inside of a wall, you push it back according to the direction its velocity has or just the direction of the character itself.

##### How it works

This works when you treat the `x` and `y` axis separately, updating one, checking the collisions that come up from it, update the other axis and check for new collisions.

\code{collisiondetection/direction_velocity}{Code for the direction + velocity collision reaction}

##### Analysis

Let's see how this method reacts in each situation.

When we are trying to fall on the ground, this method works as follows:

![How the direction + velocity method reacts to collisions on a horizontal plane](./images/collision_detection/direction_velocity_reference.png){width=80%}

1. We divide the movement vector in its `x` and `y` components.
2. We move along the `x` axis and check for collisions, in this case there are none (the ghost represents our previous position.
3. We move along the `y` axis, after checking for collisions we find that we are colliding on the ground (the ghost represents our next position).
4. We react to the collision by moving the sprite on top of the ground.

##### Quirks and issues

This method can be used only with completely solid platforms. If you want to make use of platforms that you can cross one-way, since you may get teleported around when your velocity changes direction.

![How velocity changing direction can teleport you](./images/collision_detection/velocity_teleport.png){width=80%}

In the previous example we try to jump on a platform by going through it, but our jump quite doesn't make it. Since velocity has changed direction, we end up being teleported over the platform, which is considered a glitch.

#### Shallow-axis based reaction method

This method works in a similar fashion to the direction and velocity method, but prioritizes reactions on the axis that shows the shallowest overlap.

This requires measuring how much the objects overlap on each axis, which can be a little more involved, but not really expensive.

![Example of shallow-axis based reaction](./images/collision_detection/shallow_axis.png){width=90%}

In the previous picture, we can see how the algorithm chooses to solve the collision on the $y$ axis first and only on the x axis after; but since solving the $y$ axis solves the collision, no reaction is performed on the $x$ axis.

\placeholder

<!-- TODO: Similar to direction + velocity, but reacts only on the most shallow direction -->

#### The "Snapshot" Method

This method is a bit more involved, but allows for a finer control over how you go through or collide with certain obstacles.

The secret to this method is taking a snapshot of the object's position before its update phase and do a series of comparisons with the position after the update.

\code{collisiondetection/snapshot_reaction}{Example of the "snapshot" collision reaction method}

This method solves the problem given by platforms that can be crossed one-way.

<!-- TODO: Snapshot before and after updating, react accordingly, allows for more advanced stuff -->

#### The "Tile + Offset" Method

\placeholder

<!-- TODO: Useful for games like pacman, check the direction where you are going using the offset, if the next cell is a wall, react -->

Common Issues with Collision Detection
----------------------------------------

### The "Bullet Through Paper" problem {#bulletthroughpaper}

The "bullet through paper" is a common problem with collision detection, when an obstacle is really thin (our "paper"), and the object is really fast and small (the "bullet") it can happen that collision is not detected.

![Example of the "Bullet through paper" problem](./images/collision_detection/Bullet_Through_Paper.pdf){width=40%}

The object is going so fast that it manages to go through the entirety of the obstacle in a single frame.

Possible solutions to this problems are various, some even going out of the realm of the so-called "time-stepping techniques" (like speculative contacts or ray casting) that can be very expensive from a computational standpoint.

Such solutions should therefore be enabled (or implemented) only for fast-moving objects and only if necessary, since resources and time are at a premium in most cases.

### Precision Issues

Sometimes it can happen that the position is reset incorrectly due to machine precision or wrong rounding, this can lead to the character that looks spazzy or just going through the floor at random times. The solution to these issues is making sure that the position and state are set correctly so that there are no useless state changes between frames.

Sometimes the "spazziness" of the character derives from the fact that collision reaction sets the character one pixel over the floor, triggering the "falling" state, the next frame the state would be changed to "idle" and then in the frame "n+2" the cycle would restart with collision reaction putting the character one pixel over the floor.
