# physics-simulator for testing 2d physics

## setup
First create a basic java app

```bash
mkdir physics-simulator
cd physics-simulator
gradle init

Select type of project to generate:
  1: basic
  2: groovy-application
  3: groovy-library
  4: java-application
  5: java-library
  6: kotlin-application
  7: kotlin-library
  8: scala-library
Enter selection (default: basic) [1..8] 4

Select build script DSL:
  1: groovy
  2: kotlin
Enter selection (default: groovy) [1..2] 1

Select test framework:
  1: junit
  2: testng
  3: spock
Enter selection (default: junit) [1..3] 1

Project name (default: physics-simulator): 

Source package (default: physics.simulator): 
```

Open the app in your IDE and delete the unit test and greeting stuff
leaving a simple main function.

```java
/*
 * This Java source file was generated by the Gradle 'init' task.
 */
package physics.simulator;

public class App {

    public static void main(String[] args) {
        System.out.println("hi");
    }
}

```

### Add libgdx dependencies
Open up build.gradle and add in the libgdx dependencies

```gradle
ext {
    gdxVersion = '1.9.5'
}

dependencies {

    compile "org.slf4j:slf4j-simple:1.7.25"
    compile "com.badlogicgames.gdx:gdx-backend-lwjgl:$gdxVersion"
    compile "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-desktop"
    compile "com.badlogicgames.gdx:gdx-tools:$gdxVersion"
    compile "com.badlogicgames.gdx:gdx-controllers-platform:$gdxVersion:natives-desktop"
    compile "com.badlogicgames.gdx:gdx-box2d:$gdxVersion"
    compile "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-desktop"

}
```

### Create a new Game class
We need a class to put our main game loop for the physics simulator in. Create PhysicsSimnulator.java and make it extend Game from libgdx.

```java
package physics.simulator;

import com.badlogic.gdx.Game;

public class PhysicsSimulator extends Game {

    @Override
    public void create() {

    }

    @Override
    public void render() {
        Gdx.gl.glClearColor(0, 0, 0, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
    }

}
```

### Create an empty window
Open App.java and add in some boilerplate code to the main funcrtion to create a simple render window using the PhysicsSimulator class we just created.

```java
    public static void main(String[] args) {
        LwjglApplicationConfiguration config = new LwjglApplicationConfiguration();
        config.width = 1440;
        config.height = 900;
        config.foregroundFPS = 60;

        new LwjglApplication(new PhysicsSimulator(), config);
    }

```

### Test it!
Now is a good time to run the app

## Cameras, stages and worlds
To start our physics simulation, we need to build a world that all the physics stuff happens in. Create a new world member and instantiate it in PhysicsSimulator.java

### World

```java
    private World world;

    @Override
    public void create() {

        // gravity is about 10m/s/s :)
        float gravity = 10.0f;
        world = new World(new Vector2(0, gravity), true);
        ... leave the rest of the create function alone
```

During the render call, make sure your world updates all the physics objects it will eventually manage.

```java
    public void render() {
        ... above is the clear and such
        // put this at the end
        world.step(Gdx.graphics.getDeltaTime(), 6, 2);
    }
```

### Stages and Cameras
libgdx has the concept of stages with actors and a camera viewing the stage. For our purposes, we will have an empty stage and just use the physics debug renderer, but we still need a stage for the camera to point at.

Create two more member variables for the camera and stage
```java
    private OrthographicCamera camera;
    private Stage stage;
```

Configure the camera in the create function. This is a bit funny. We want our camera to view a 10x10 meter space in the world, but we also want to make sure it keeps the same aspect ratio as our display. As such, keep the width at 10, but adjust the height to match what it would be with the same aspect ratio as the display window.

Create a viewport to hold our camera, and assign that viewpor to a stage

```java
        // make our camera view a 10x10 meter space
        float viewWidth = 10;
        float viewHeight = 10;

        // make sure the aspect ratio matches our display window
        camera = new OrthographicCamera();
        camera.setToOrtho(false, viewWidth, viewHeight * ((float)Gdx.graphics.getHeight() / Gdx.graphics.getWidth()));

        // Put the camera at the origin
        camera.translate(0f, 0f);
        camera.update();

        // create a viewport to fit the display window, and use
        // our camera
        Viewport viewport = new FitViewport(viewWidth, viewHeight * ((float)Gdx.graphics.getHeight() / Gdx.graphics.getWidth()), camera);

        // Create a new stage with this viewport
        stage = new Stage(viewport);
```

Next, in the render function, update our stage.

```java
    @Override
    public void render() {
        // clear the window to black
        Gdx.gl.glClearColor(0, 0, 0, 1);
        Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);

        camera.update();
        
        stage.act();
        stage.draw();

        world.step(Gdx.graphics.getDeltaTime(), 6, 2);
    }
```

### Debug Renderer
The Box2D physics engine comes with a handy debug renderer so we can see how our various objects are interacting in the world without having to worry about graphics or textures. Let's create one.

```java
    private Box2DDebugRenderer debugRenderer;

    @Override
    public void create() {
        debugRenderer = new Box2DDebugRenderer();
        ...
```

in render(), before we update the physics world, render the debug stuff

```java
        ....
        debugRenderer.render(world, stage.getCamera().combined);
        world.step(Gdx.graphics.getDeltaTime(), 6, 2);
    }
```

If you launch at this point, it should work, but we won't see anything.

### Create a box
Let's create some objects

## Moving the camera
```java
        // in create
        Gdx.input.setInputProcessor(stage);

        // in render
        if (Gdx.input.isKeyPressed(Input.Keys.PLUS)) {
            camera.zoom += 0.02;
        }
        if (Gdx.input.isKeyPressed(Input.Keys.MINUS)) {
            camera.zoom -= 0.02;
        }
        if (Gdx.input.isKeyPressed(Input.Keys.LEFT)) {
            camera.translate(-1.0f*delta, 0, 0);
        }
        if (Gdx.input.isKeyPressed(Input.Keys.RIGHT)) {
            camera.translate(1*delta, 0, 0);
        }
        if (Gdx.input.isKeyPressed(Input.Keys.DOWN)) {
            camera.translate(0, -1*delta, 0);
        }
        if (Gdx.input.isKeyPressed(Input.Keys.UP)) {
            camera.translate(0, 1*delta, 0);
        }

        if (Gdx.input.isTouched()) {
            Vector3 mouseOnScreen2D = new Vector3();
            mouseOnScreen2D.x = Gdx.input.getX();
            mouseOnScreen2D.y = Gdx.input.getY();

            Vector3 mouseInWorldCoords = camera.unproject(mouseOnScreen2D);
            body.resetToPosition(mouseInWorldCoords.x, mouseInWorldCoords.y);
        }        
```

