# ODGraphics Engine Guide

A comprehensive guide for using the ODGraphics 3D rendering engine to create games and applications.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Creating a Window](#creating-a-window)
3. [The Rendering Pipeline](#the-rendering-pipeline)
4. [Working with Models](#working-with-models)
5. [Materials and PBR](#materials-and-pbr)
6. [Lighting System](#lighting-system)
7. [Camera Controls](#camera-controls)
8. [Text Rendering](#text-rendering)
9. [Physics System](#physics-system)
10. [Render Settings](#render-settings)
11. [Post-Processing Effects](#post-processing-effects)
12. [ImGui Integration](#imgui-integration)
13. [Loading 3D Models](#loading-3d-models)
14. [Complete Example](#complete-example)

---

## Getting Started

### Maven Dependency

Add RedEngine to your project by including the dependency in your `pom.xml`:

**Option 1: Platform-specific JAR with bundled natives (Recommended)**

Choose the classifier for your target platform:

```xml
<!-- Windows x64 -->
<dependency>
    <groupId>io.redstonerdev</groupId>
    <artifactId>RedEngine</artifactId>
    <version>0.6.0</version>
    <classifier>windows-x64</classifier>
</dependency>

<!-- Linux x64 -->
<dependency>
    <groupId>io.redstonerdev</groupId>
    <artifactId>RedEngine</artifactId>
    <version>0.6.0</version>
    <classifier>linux-x64</classifier>
</dependency>

<!-- macOS x64 -->
<dependency>
    <groupId>io.redstonerdev</groupId>
    <artifactId>RedEngine</artifactId>
    <version>0.6.0</version>
    <classifier>macos-x64</classifier>
</dependency>
```

**Option 2: Core library (provide your own natives)**

If you want to manage native dependencies yourself:

```xml
<dependency>
    <groupId>io.redstonerdev</groupId>
    <artifactId>RedEngine</artifactId>
    <version>0.6.0</version>
</dependency>
```

With this option, you must also add LWJGL and ImGui native dependencies for your target platform(s).

### Dependencies

The engine uses the following libraries:
- **LWJGL 3** - OpenGL bindings for Java
- **JOML** - Java OpenGL Math Library
- **STB** - Font and image loading
- **ImGui-Java** - Immediate mode GUI
- **ODE4J** - Physics engine (Open Dynamics Engine for Java)

### Basic Structure

Every application extends the `Window` class and follows this pattern:

```java
public class MyGame extends Window {

    public MyGame() throws Exception {
        super(0, 0, 1280, 720);  // x, y, width, height
        setTitle("My Game");

        // Initialize your game
        init();

        // Start the game loop
        loop();
    }

    public static void main(String[] args) throws Exception {
        new MyGame();
    }
}
```

---

## Creating a Window

### Window Configuration

```java
public class MyGame extends Window {

    public MyGame() throws Exception {
        super(0, 0, 1920, 1080);  // Position and size

        // Window settings
        setTitle("My Awesome Game");
        setDebugMode(true);  // Enable debug info

        // Frame rate control
        getLimiter().setMode(FrameLimiter.Mode.LIMITED);
        getLimiter().setTargetFps(144);

        // Or use unlimited/vsync
        // getLimiter().setMode(FrameLimiter.Mode.UNLIMITED);
        // getLimiter().setMode(FrameLimiter.Mode.VSYNC);
    }
}
```

### MSAA (Anti-Aliasing) Configuration

MSAA must be configured **before** window creation for best results:

```java
public class MyGame extends Window {

    public MyGame() throws Exception {
        // Set MSAA BEFORE calling super constructor or init()
        Window.setMsaaSamples(4);  // 2, 4, 8, or 16 samples

        super(0, 0, 1920, 1080);
        // ...
    }
}

// Get current MSAA setting
int samples = Window.getMsaaSamples();
```

**Note:** The post-processing system automatically uses multisampled framebuffers when MSAA is enabled. MSAA can also be changed at runtime through RenderSettings, which will update the PostProcessing framebuffers.

```java
// Runtime MSAA change (updates post-processing framebuffers)
RenderSettings.getInstance().setAntiAliasing(RenderSettings.AntiAliasing.MSAA_8X);
PostProcessing.getInstance().setMsaaSamples(8);
```
```

### Mouse Capture (for FPS-style games)

```java
// Lock and hide the cursor
GLFW.glfwSetInputMode(getWindow(), GLFW.GLFW_CURSOR, GLFW.GLFW_CURSOR_DISABLED);

// Release the cursor
GLFW.glfwSetInputMode(getWindow(), GLFW.GLFW_CURSOR, GLFW.GLFW_CURSOR_NORMAL);
```

---

## The Rendering Pipeline

### Adding Renderers

The engine uses a callback-based rendering system:

```java
addRenderer(new RenderI() {
    @Override
    public void render(Renderer renderer, Renderer3D renderer3D, TextRenderer textRenderer) {
        // Render 3D models
        renderer3D.renderModels(myModels);

        // Render 2D text overlay
        textRenderer.renderText("Score: 100", 10, 10, 24, Color.WHITE,
            renderer.getWidth(), renderer.getHeight());
    }
});
```

### Render Order

1. **3D Scene** - Models, lighting, shadows
2. **Skybox/Atmosphere** - Background rendering
3. **Post-Processing** - Bloom, tone mapping
4. **2D Overlay** - Text, HUD elements
5. **ImGui** - Debug UI

---

## Working with Models

### Creating Procedural Meshes

The `Shapes` class provides factory methods for creating common 3D primitives:

```java
import io.redstonerdev.threed.model.Shapes;
import io.redstonerdev.threed.model.Mesh;
import io.redstonerdev.threed.model.Model;

// Create a cube (fixed size 1x1x1)
Model cube = Shapes.createCube(new Vector3f(1, 0, 0));  // Red cube

// Create a plane/ground
Model ground = Shapes.createPlane(20.0f, new Vector3f(0.5f, 0.5f, 0.5f));  // Gray 20x20 plane

// Create a sphere
Model sphere = Shapes.createSphere(0.5f, 32, 32, new Vector3f(0, 1, 0));  // Green sphere

// Or get just the mesh and create the model yourself with a Material:
Mesh cubeMesh = Shapes.createCubeMesh();
Mesh planeMesh = Shapes.createPlaneMesh(20.0f);
Mesh sphereMesh = Shapes.createSphereMesh(0.5f, 32, 32);
Mesh cylinderMesh = Shapes.createCylinderMesh(0.5f, 2.0f, 32);

Model goldCube = new Model(cubeMesh, Material.createGold());
```

### Model Transformations

```java
Model cube = createCube(1.0f, new Vector3f(1, 0, 0));

// Position
cube.setPosition(new Vector3f(0, 5, -10));

// Rotation (in degrees)
cube.setRotation(new Vector3f(45, 30, 0));  // pitch, yaw, roll

// Scale
cube.setScale(new Vector3f(2, 1, 2));  // width, height, depth

// Incremental rotation
cube.rotate(1.0f, 0.5f);  // yaw, pitch in degrees
```

### Model Hierarchies (Parent-Child)

```java
Model parent = createCube(1.0f, new Vector3f(1, 1, 1));
Model child = createCube(0.5f, new Vector3f(1, 0, 0));

// Add child - it will inherit parent's transformations
parent.addChild("arm", child);
child.setPosition(new Vector3f(2, 0, 0));  // Offset from parent

// Access children
Model arm = parent.getChild("arm");
List<Model> allChildren = parent.getChildren();
```

---

## Materials and PBR

The engine supports Physically Based Rendering (PBR) materials.

### Basic Material Creation

```java
// Simple colored material
Material mat = Material.fromColor(new Vector3f(1, 0, 0));  // Red

// Material from texture
Texture tex = Texture.loadTexture("textures/wood.png");
Material mat = Material.fromTexture(tex);
```

### PBR Material Properties

```java
Material mat = new Material();

// Base color (albedo)
mat.setAlbedoFactor(1.0f, 0.8f, 0.6f, 1.0f);  // RGBA
mat.setAlbedoMap(albedoTexture);

// Metallic-Roughness
mat.setMetallicFactor(0.0f);   // 0 = dielectric, 1 = metal
mat.setRoughnessFactor(0.5f);  // 0 = mirror, 1 = rough
mat.setMetallicRoughnessMap(mrTexture);

// Normal mapping
mat.setNormalMap(normalTexture);
mat.setNormalScale(1.0f);

// Ambient Occlusion
mat.setOcclusionMap(aoTexture);
mat.setOcclusionStrength(1.0f);

// Emissive (glow)
mat.setEmissiveFactor(1.0f, 0.5f, 0.0f);  // RGB
mat.setEmissiveMap(emissiveTexture);

// Transparency
mat.setAlphaMode(Material.AlphaMode.BLEND);
mat.setAlphaCutoff(0.5f);  // For MASK mode
```

### Pre-built Material Types

```java
// Metals
Material gold = Material.createGold();
Material silver = Material.createSilver();
Material copper = Material.createCopper();
Material steel = Material.createBrushedSteel();

// Custom metallic
Material chrome = Material.createMetallic(
    new Vector3f(0.9f, 0.9f, 0.9f),  // color
    1.0f,   // metallic
    0.1f    // roughness
);

// Emissive (glowing)
Material glow = Material.createEmissive(
    new Vector3f(1, 0.5f, 0),  // emission color
    3.0f                        // intensity (>1 for bloom)
);

// Emissive with base color
Material neonSign = Material.createEmissive(
    new Vector3f(0.1f, 0.1f, 0.1f),  // base color
    new Vector3f(0, 1, 0),            // emission color
    5.0f                              // intensity
);

// Glass/Transparent
Material glass = Material.createGlass(
    new Vector3f(1, 1, 1),  // tint
    0.2f                     // opacity
);
Material clearGlass = Material.createClearGlass();

// Plastic
Material plastic = Material.createPlastic(
    new Vector3f(1, 0, 0),  // color
    0.4f                     // roughness
);
```

### Applying Materials to Models

```java
Model cube = new Model(mesh, Material.createGold());

// Or set later
cube.setMaterial(Material.createEmissive(new Vector3f(1, 0, 0), 2.0f));
```

---

## Lighting System

### Light Types

Lights are created using factory methods:

```java
Renderer3D renderer3D = Renderer3D.getInstance();

// Directional Light (Sun) - use factory method
Light sun = Light.createDirectional(
    new Vector3f(1, -1, 0.5f),    // direction
    new Vector3f(1, 1, 0.9f),     // color
    1.0f                          // intensity
);
renderer3D.addLight(sun);

// Point Light (Bulb)
Light bulb = Light.createPoint(
    new Vector3f(0, 3, 0),        // position
    new Vector3f(1, 0.8f, 0.6f),  // color
    2.0f                          // intensity
);
bulb.setAttenuation(1.0f, 0.09f, 0.032f);  // constant, linear, quadratic
renderer3D.addLight(bulb);

// Spot Light (Flashlight)
Light spot = Light.createSpot(
    new Vector3f(0, 5, 0),        // position
    new Vector3f(0, -1, 0),       // direction
    new Vector3f(1, 1, 1),        // color
    3.0f                          // intensity
);
spot.setSpotlightAngles(12.5f, 17.5f);  // inner and outer cone angles
renderer3D.addLight(spot);

// You can also modify properties after creation:
sun.setDirection(new Vector3f(1, -2, 1));
sun.setIntensity(1.5f);
bulb.setPosition(new Vector3f(5, 3, 0));
```

### Managing Lights

```java
// Get all lights
List<Light> lights = renderer3D.getLights();

// Get specific light
Light light = renderer3D.getLight(0);

// Remove light
renderer3D.removeLight(light);

// Clear all lights
renderer3D.clearLights();

// Toggle light
light.setEnabled(false);
```

### Sun Direction (Time of Day)

```java
// Set sun direction based on time
float timeOfDay = 0.5f;  // 0-1, where 0.5 is noon
float angle = (timeOfDay - 0.5f) * Math.PI;
Vector3f sunDir = new Vector3f(
    (float) Math.cos(angle),
    (float) -Math.abs(Math.sin(angle)),
    0.3f
).normalize();

renderer3D.setLightDir(sunDir);
```

---

## Camera Controls

### First-Person Camera

The Camera class provides direct control over position and orientation via yaw/pitch angles:

```java
Camera camera = Camera.getInstance();

// Set initial position
camera.setPosition(new Vector3f(0, 2, 10));

// Mouse look - update yaw and pitch directly
private double lastMouseX, lastMouseY;
private float sensitivity = 0.1f;

glfwSetCursorPosCallback(getWindow(), (window, xpos, ypos) -> {
    float deltaX = (float) (xpos - lastMouseX) * sensitivity;
    float deltaY = (float) (ypos - lastMouseY) * sensitivity;

    // Update yaw (horizontal) and pitch (vertical) directly
    camera.setYaw(camera.getYaw() + deltaX);
    camera.setPitch(camera.getPitch() - deltaY);

    // Clamp pitch to avoid flipping
    if (camera.getPitch() > 89.0f) camera.setPitch(89.0f);
    if (camera.getPitch() < -89.0f) camera.setPitch(-89.0f);

    lastMouseX = xpos;
    lastMouseY = ypos;
});

// WASD Movement - use direction vectors for movement
private void handleMovement(float deltaTime) {
    float speed = 5.0f * deltaTime;
    Camera camera = Camera.getInstance();

    // Get the direction the camera is facing (normalized)
    Vector3f direction = camera.getDirection();

    // Calculate right vector (perpendicular to direction on XZ plane)
    Vector3f right = new Vector3f(direction.z, 0, -direction.x).normalize();

    // Forward/backward (along camera direction, but only on XZ plane for FPS)
    Vector3f forward = new Vector3f(direction.x, 0, direction.z).normalize();

    Vector3f movement = new Vector3f();

    if (isKeyPressed(GLFW_KEY_W)) movement.add(forward.mul(speed, new Vector3f()));
    if (isKeyPressed(GLFW_KEY_S)) movement.sub(forward.mul(speed, new Vector3f()));
    if (isKeyPressed(GLFW_KEY_A)) movement.sub(right.mul(speed, new Vector3f()));
    if (isKeyPressed(GLFW_KEY_D)) movement.add(right.mul(speed, new Vector3f()));
    if (isKeyPressed(GLFW_KEY_SPACE)) movement.y += speed;
    if (isKeyPressed(GLFW_KEY_LEFT_SHIFT)) movement.y -= speed;

    camera.move(movement);
}
```

### Camera Properties

```java
Camera camera = Camera.getInstance();

// Get camera matrices
Matrix4f viewMatrix = camera.getViewMatrix();
Matrix4f projMatrix = Renderer3D.getInstance().getProjectionMatrix();

// Position
Vector3f position = camera.getPosition();
camera.setPosition(new Vector3f(0, 5, 10));

// Orientation (in degrees)
float yaw = camera.getYaw();      // Horizontal rotation
float pitch = camera.getPitch();  // Vertical rotation
camera.setYaw(180.0f);
camera.setPitch(0.0f);

// Get the direction the camera is looking
Vector3f direction = camera.getDirection();  // Normalized forward vector

// Move camera by an offset
camera.move(new Vector3f(1, 0, 0));  // Move 1 unit to the right
```

---

## Text Rendering

### 2D Text (Screen Space)

```java
TextRenderer textRenderer = TextRenderer.getInstance();

// Basic text
textRenderer.renderText("Hello World", 10, 10, 24, Color.WHITE,
    screenWidth, screenHeight);

// Colored text
textRenderer.renderText("Health: 100", 10, 40, 20, Color.RED,
    screenWidth, screenHeight);

// With transparency
Color semiTransparent = new Color(255, 255, 255, 128);
textRenderer.renderText("Fading...", 10, 70, 18, semiTransparent,
    screenWidth, screenHeight);
```

### 3D Text (World Space)

```java
Matrix4f projection = renderer3D.getProjectionMatrix();
Matrix4f view = camera.getViewMatrix();

// Billboard text (always faces camera)
textRenderer.renderText3D(
    "Player Name",
    new Vector3f(0, 3, 0),  // world position
    32,                      // font size
    1.0f,                    // scale
    Color.WHITE,
    projection, view
);

// With distance fade
textRenderer.renderText3DWithFade(
    "Distant Label",
    new Vector3f(100, 5, 0),
    32, 1.0f, Color.WHITE,
    camera.getPosition(),
    50.0f,   // fade start distance
    100.0f,  // fade end distance
    projection, view
);

// Constant screen size (doesn't shrink with distance)
textRenderer.renderText3DConstantSize(
    "Always Same Size",
    new Vector3f(0, 5, 0),
    32, 1.0f, Color.WHITE,
    camera.getPosition(),
    10.0f,  // reference distance
    projection, view
);
```

### SDF Text Settings

```java
// Enable/disable SDF for 3D text (sharper at close range)
textRenderer.setUseSdfFor3D(true);

// Adjust edge smoothness
textRenderer.setSdfSmoothing(0.1f);

// Add outline
textRenderer.setSdfOutlineWidth(0.1f);
textRenderer.setSdfOutlineColor(new Vector3f(0, 0, 0));
```

---

## Physics System

The engine includes a physics system powered by ODE4J (Open Dynamics Engine for Java). It provides rigid body dynamics, collision detection, and physics simulation.

### PhysicsWorld

The `PhysicsWorld` is a singleton that manages all physics simulation.

```java
// Get the physics world instance
PhysicsWorld world = PhysicsWorld.getInstance();

// Configure physics settings
world.setGravity(new Vector3f(0, -9.81f, 0));  // Earth gravity
world.setSolverIterations(10);  // Higher = more stable but slower

// Default material properties
world.setDefaultFriction(0.5f);
world.setDefaultBounce(0.3f);
```

### Creating Physics Bodies

#### Dynamic Bodies (affected by physics)

```java
// Create a dynamic sphere (1 kg mass)
SphereCollider sphereShape = new SphereCollider(0.5f);  // radius 0.5
PhysicsBody ball = world.createDynamicBody(sphereShape, 1.0f);
ball.setPosition(new Vector3f(0, 10, 0));
ball.setRestitution(0.7f);  // Bouncy
ball.setFriction(0.3f);

// Create a dynamic box
BoxCollider boxShape = new BoxCollider(new Vector3f(0.5f, 0.5f, 0.5f));  // half-extents
PhysicsBody box = world.createDynamicBody(boxShape, 2.0f);  // 2 kg
box.setPosition(new Vector3f(0, 5, 0));

// Create a dynamic capsule (good for characters)
CapsuleCollider capsuleShape = new CapsuleCollider(0.3f, 1.0f);  // radius, height
PhysicsBody character = world.createDynamicBody(capsuleShape, 70.0f);  // 70 kg
```

#### Static Bodies (immovable)

```java
// Create a static box (floor, walls, etc.)
BoxCollider floorShape = new BoxCollider(new Vector3f(50, 0.5f, 50));
PhysicsBody floor = world.createStaticBody(floorShape);
floor.setPosition(new Vector3f(0, -0.5f, 0));

// Create an infinite ground plane (most efficient for flat ground)
PhysicsBody ground = world.createGroundPlane(0);  // Y = 0
```

### Collider Types

```java
// Sphere - Fastest collision detection
SphereCollider sphere = new SphereCollider(radius);
SphereCollider sphereOffset = new SphereCollider(radius, localCenterOffset);

// Box (OBB) - Supports rotation
BoxCollider box = new BoxCollider(halfExtents);
BoxCollider boxFromBounds = BoxCollider.fromMinMax(min, max);

// Capsule - Good for characters, pill-shaped
CapsuleCollider capsule = new CapsuleCollider(radius, height);
CapsuleCollider capsuleX = new CapsuleCollider(radius, height, CapsuleCollider.Axis.X);

// Plane - Infinite, static only
PlaneCollider plane = new PlaneCollider(normal, distance);
PlaneCollider ground = PlaneCollider.ground(yPosition);
```

### Body Types

```java
public enum BodyType {
    STATIC,    // Immovable (terrain, walls)
    DYNAMIC,   // Fully simulated (physics objects)
    KINEMATIC  // Code-controlled, affects dynamics (moving platforms)
}

// Check body type
if (body.isDynamic()) { ... }
if (body.isStatic()) { ... }
```

### Applying Forces and Impulses

```java
PhysicsBody body = ...;

// Apply continuous force (use in update loop)
body.applyForce(new Vector3f(0, 100, 0));  // Upward force
body.applyForceAtPosition(force, worldPoint);  // Force at specific point

// Apply torque (rotational force)
body.applyTorque(new Vector3f(0, 10, 0));  // Spin around Y axis

// Apply instant impulse (for jumping, explosions)
body.applyImpulse(new Vector3f(0, 5, 0));  // Instant velocity change

// Direct velocity control
body.setLinearVelocity(new Vector3f(0, 10, 0));
body.setAngularVelocity(new Vector3f(0, 5, 0));
```

### Physics Body Properties

```java
// Transform
Vector3f pos = body.getPosition();
body.setPosition(new Vector3f(0, 5, 0));

Quaternionf rot = body.getRotation();
body.setRotation(rotation);

Vector3f euler = body.getRotationEuler();  // Degrees
body.setRotationEuler(new Vector3f(0, 45, 0));

// Velocity
Vector3f linearVel = body.getLinearVelocity();
Vector3f angularVel = body.getAngularVelocity();

// Material properties
body.setFriction(0.5f);      // 0 = ice, 1 = rubber
body.setRestitution(0.7f);   // 0 = no bounce, 1 = perfect bounce

// Sleep state (optimization for resting objects)
body.isSleeping();  // Check if asleep
body.wakeUp();      // Wake up the body

// Enable/disable
body.setEnabled(false);  // Disable physics
body.isEnabled();

// User data (attach game objects)
body.setUserData(myGameObject);
Object data = body.getUserData();
```

### Updating Physics

```java
// In your game loop
@Override
protected void update(float deltaTime) {
    // Scale deltaTime if needed for slow-motion effects
    float physicsTimeScale = 0.02f;  // Adjust as needed
    float dt = deltaTime * physicsTimeScale;

    // Update physics simulation
    PhysicsWorld.getInstance().update(dt);

    // Sync model transforms to physics bodies
    for (Map.Entry<Model, PhysicsBody> entry : modelToBody.entrySet()) {
        Model model = entry.getKey();
        PhysicsBody body = entry.getValue();

        model.setPosition(body.getPosition());
        model.setRotation(body.getRotationEuler());
    }
}
```

### Cleanup

```java
// Remove a single body
world.removeBody(body);

// Reset entire physics world
world.reset();

// Full cleanup (call on shutdown)
world.destroy();
```

### Complete Physics Example

```java
import io.redstonerdev.gui.window.Window;
import io.redstonerdev.threed.model.Material;
import io.redstonerdev.threed.model.Model;
import io.redstonerdev.threed.model.Shapes;
import io.redstonerdev.threed.physics.PhysicsBody;
import io.redstonerdev.threed.physics.PhysicsWorld;
import io.redstonerdev.threed.physics.SphereCollider;
import org.joml.Vector3f;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PhysicsDemo extends Window {
    private PhysicsWorld physicsWorld;
    private List<Model> models = new ArrayList<>();
    private Map<Model, PhysicsBody> modelToBody = new HashMap<>();

    public PhysicsDemo() throws Exception {
        super(0, 0, 1280, 720);
        setTitle("Physics Demo");

        // Initialize physics
        physicsWorld = PhysicsWorld.getInstance();
        physicsWorld.setGravity(new Vector3f(0, -9.81f, 0));

        // Create ground
        PhysicsBody ground = physicsWorld.createGroundPlane(0);
        ground.setFriction(0.5f);

        Model groundModel = new Model(Shapes.createPlaneMesh(50),
            Material.createPlastic(new Vector3f(0.4f), 0.8f));
        models.add(groundModel);

        // Create falling spheres
        for (int i = 0; i < 10; i++) {
            // Physics body
            SphereCollider shape = new SphereCollider(0.5f);
            PhysicsBody body = physicsWorld.createDynamicBody(shape, 1.0f);
            body.setPosition(new Vector3f(i * 1.5f - 7, 10 + i * 2, 0));
            body.setRestitution(0.6f);

            // Visual model
            Model sphere = new Model(Shapes.createSphereMesh(0.5f, 32, 32),
                Material.createGold());
            models.add(sphere);
            modelToBody.put(sphere, body);
        }

        // Set up rendering
        addRenderer((r, r3d, tr) -> r3d.renderModels(models));

        init();
        loop();
    }

    @Override
    protected void update(float deltaTime) {
        // Update physics
        physicsWorld.update(deltaTime * 0.02f);

        // Sync transforms
        for (Map.Entry<Model, PhysicsBody> entry : modelToBody.entrySet()) {
            entry.getKey().setPosition(entry.getValue().getPosition());
            entry.getKey().setRotation(entry.getValue().getRotationEuler());
        }
    }
}
```

---

## Render Settings

### Accessing Settings

```java
RenderSettings settings = RenderSettings.getInstance();
```

### Quality Presets

```java
// Apply a preset
settings.applyPreset(RenderSettings.QualityPreset.HIGH);

// Available presets: LOW, MEDIUM, HIGH, ULTRA
```

### Shadow Settings

```java
settings.setShadowsEnabled(true);
settings.setShadowQuality(RenderSettings.ShadowQuality.HIGH);

// Fine-tune
settings.setShadowBiasMin(0.002f);
settings.setShadowBiasMax(0.01f);
settings.setShadowSoftness(1.0f);
settings.setShadowOrthoSize(50.0f);
```

### PBR / Post-Processing

```java
settings.setPbrEnabled(true);
settings.setToneMapping(true);
settings.setToneMappingOperator(RenderSettings.ToneMappingOperator.ACES);
settings.setExposure(1.0f);
settings.setGamma(2.2f);
```

### Anti-Aliasing

```java
// Set MSAA quality
settings.setAntiAliasing(RenderSettings.AntiAliasing.MSAA_4X);

// Available options: OFF, MSAA_2X, MSAA_4X, MSAA_8X, MSAA_16X

// Get current sample count
int samples = settings.getMsaaSamples();
```

**Note:** Changing MSAA via RenderSettings automatically updates the PostProcessing framebuffers.

### Camera Settings

```java
settings.setFov(90.0f);
settings.setNearPlane(0.01f);
settings.setFarPlane(1000.0f);
```

### Debug Options

```java
settings.setWireframeMode(true);
settings.setShowNormals(true);
settings.setShowShadowMap(true);
```

---

## Post-Processing Effects

The post-processing system handles HDR rendering, bloom, tone mapping, and MSAA integration.

### PostProcessing System

The `PostProcessing` singleton manages framebuffers and post-processing effects:

```java
PostProcessing pp = PostProcessing.getInstance();

// Initialize/resize framebuffers (called automatically by Window)
pp.resize(screenWidth, screenHeight);

// Begin capturing scene to HDR framebuffer
pp.beginSceneCapture();
// ... render your scene here ...
pp.endSceneCapture();

// Apply bloom and render to screen
pp.applyBloom();
```

### MSAA Integration

The post-processing system uses multisampled framebuffers when MSAA is enabled. The pipeline:

1. Scene is rendered to a multisampled FBO (GL_TEXTURE_2D_MULTISAMPLE)
2. Multisampled buffer is resolved to a regular texture (glBlitFramebuffer)
3. Post-processing effects (bloom) are applied to the resolved texture
4. Final result is rendered to screen with tone mapping

```java
// Set MSAA samples at runtime
PostProcessing.getInstance().setMsaaSamples(4);  // 1, 2, 4, 8, or 16

// Check current MSAA setting
int samples = PostProcessing.getInstance().getMsaaSamples();
```

### Bloom

Bloom makes bright areas glow, perfect for lights and emissive materials.

```java
RenderSettings settings = RenderSettings.getInstance();

// Enable bloom
settings.setBloomEnabled(true);

// Quality preset
settings.setBloomQuality(RenderSettings.BloomQuality.MEDIUM);

// Fine-tune
settings.setBloomThreshold(1.0f);      // Brightness cutoff
settings.setBloomSoftThreshold(0.5f);  // Soft transition
settings.setBloomIntensity(1.0f);      // Glow strength
settings.setBloomBlurPasses(5);        // Blur iterations
settings.setBloomBlurScale(1.0f);      // Blur spread
```

### Tone Mapping

HDR to LDR conversion with various operators:

```java
settings.setToneMapping(true);
settings.setToneMappingOperator(RenderSettings.ToneMappingOperator.ACES);
settings.setExposure(1.0f);  // HDR exposure
settings.setGamma(2.2f);     // Gamma correction
```

Available tone mapping operators:
- `REINHARD` - Classic, balanced
- `ACES` - Filmic, cinematic look
- `UNCHARTED2` - Filmic with more contrast
- `EXPOSURE` - Simple exposure-based

### Making Objects Glow with Bloom

```java
// Emissive materials with intensity > 1.0 will trigger bloom
Material glowingRed = Material.createEmissive(
    new Vector3f(1, 0, 0),  // color
    3.0f                     // intensity > 1 = bloom
);

// Lights naturally cause bloom on nearby surfaces
Light brightLight = new Light(Light.Type.POINT);
brightLight.setIntensity(5.0f);  // High intensity
```

---

## ImGui Integration

### Adding ImGui Windows

```java
addImGuiRender(new ImGuiRender() {
    @Override
    public void render() {
        ImGui.begin("My Debug Window");

        ImGui.text("FPS: " + getFps());

        if (ImGui.button("Reset Position")) {
            camera.setPosition(new Vector3f(0, 0, 0));
        }

        ImGui.sliderFloat("Speed", speedArray, 0.1f, 10.0f);

        ImGui.end();
    }
});
```

### Render Settings Window

The engine includes a built-in settings UI:

```java
addImGuiRender(new ImGuiRender() {
    @Override
    public void render() {
        // Render the complete settings panel
        RenderSettings.getInstance().renderImGui("Render Settings");
    }
});
```

### Common ImGui Widgets

```java
// Checkbox
if (ImGui.checkbox("Enable Feature", booleanValue)) {
    booleanValue = !booleanValue;
}

// Slider
float[] value = {1.0f};
ImGui.sliderFloat("Value", value, 0.0f, 10.0f);

// Color picker
float[] color = {1.0f, 0.0f, 0.0f};
ImGui.colorEdit3("Color", color);

// Combo box
String[] options = {"Option A", "Option B", "Option C"};
ImInt selected = new ImInt(0);
ImGui.combo("Select", selected, options);

// Collapsing header
if (ImGui.collapsingHeader("Advanced")) {
    // Content here
}
```

---

## Loading 3D Models

### GLB/GLTF Models

```java
// Load a GLB model
Model model = GLBLoader.load("models/character.glb");

// Normalize size (fit within radius)
model.normalize(1.0f);

// Position and add to scene
model.setPosition(new Vector3f(0, 0, 0));
models.add(model);
```

### Model with Animations

```java
Model character = GLBLoader.load("models/animated_character.glb");

// Check for skeleton
if (character.hasSkeleton()) {
    // List available animations
    Map<String, SkeletalAnimation> anims = character.getAnimations();

    // Play an animation
    character.playAnimation("walk");

    // In your update loop
    character.updateAnimation(deltaTime);
}
```

---

## Complete Example

Here's a complete example of a simple game setup:

```java
import io.redstonerdev.graphics.RenderI;
import io.redstonerdev.graphics.font.TextRenderer;
import io.redstonerdev.graphics.render.Renderer;
import io.redstonerdev.gui.events.ImGuiRender;
import io.redstonerdev.gui.window.FrameLimiter;
import io.redstonerdev.gui.window.Window;
import io.redstonerdev.threed.lighting.Light;
import io.redstonerdev.threed.model.Material;
import io.redstonerdev.threed.model.Model;
import io.redstonerdev.threed.model.Shapes;
import io.redstonerdev.threed.render.Camera;
import io.redstonerdev.threed.render.Renderer3D;
import io.redstonerdev.threed.render.RenderSettings;
import org.joml.Vector3f;
import org.lwjgl.glfw.GLFW;

import java.awt.Color;
import java.util.ArrayList;
import java.util.List;

public class MyGame extends Window {

    private List<Model> models = new ArrayList<>();
    private double lastMouseX, lastMouseY;
    private boolean firstMouse = true;

    public MyGame() throws Exception {
        super(0, 0, 1920, 1080);
        setTitle("My Game");
        setDebugMode(true);

        // Frame rate
        getLimiter().setMode(FrameLimiter.Mode.LIMITED);
        getLimiter().setTargetFps(144);

        // Quality settings
        RenderSettings.getInstance().applyPreset(RenderSettings.QualityPreset.HIGH);
        RenderSettings.getInstance().setBloomEnabled(true);

        // Setup scene
        setupScene();
        setupLighting();
        setupCamera();
        setupInput();

        // Rendering
        addRenderer(new RenderI() {
            @Override
            public void render(Renderer renderer, Renderer3D renderer3D,
                               TextRenderer textRenderer) {
                renderer3D.renderModels(models);

                textRenderer.renderText(getFps() + " FPS", 10, 10, 24,
                        Color.WHITE, renderer.getWidth(), renderer.getHeight());
            }
        });

        // ImGui
        addImGuiRender(new ImGuiRender() {
            @Override
            public void render() {
                RenderSettings.getInstance().renderImGui();
            }
        });

        // Start game loop
        loop();
    }

    private void setupScene() {
        // Ground - use Shapes class
        Model ground = new Model(Shapes.createPlaneMesh(50),
                Material.createPlastic(new Vector3f(0.3f, 0.3f, 0.3f), 0.8f));
        ground.setPosition(new Vector3f(0, 0, 0));
        models.add(ground);

        // Some cubes
        for (int i = 0; i < 5; i++) {
            Model cube = new Model(Shapes.createCubeMesh(), Material.createGold());
            cube.setPosition(new Vector3f(i * 3 - 6, 0.5f, -5));
            models.add(cube);
        }

        // Glowing sphere
        Model glow = new Model(Shapes.createSphereMesh(0.5f, 32, 32),
                Material.createEmissive(new Vector3f(1, 0.5f, 0), 5.0f));
        glow.setPosition(new Vector3f(0, 2, -5));
        models.add(glow);
    }

    private void setupLighting() {
        Renderer3D renderer3D = Renderer3D.getInstance();

        // Sun - use factory method
        Light sun = Light.createDirectional(
            new Vector3f(1, -2, 1),
            new Vector3f(1, 0.95f, 0.9f),
            1.0f
        );
        renderer3D.addLight(sun);

        // Point light - use factory method
        Light point = Light.createPoint(
            new Vector3f(0, 3, -5),
            new Vector3f(1, 0.8f, 0.6f),
            2.0f
        );
        renderer3D.addLight(point);
    }

    private void setupCamera() {
        Camera camera = Camera.getInstance();
        camera.setPosition(new Vector3f(0, 2, 5));

        // Lock cursor
        GLFW.glfwSetInputMode(getWindow(), GLFW.GLFW_CURSOR,
                GLFW.GLFW_CURSOR_DISABLED);
    }

    private void setupInput() {
        // Mouse look
        GLFW.glfwSetCursorPosCallback(getWindow(), (window, xpos, ypos) -> {
            if (firstMouse) {
                lastMouseX = xpos;
                lastMouseY = ypos;
                firstMouse = false;
            }

            float sensitivity = 0.1f;
            float deltaX = (float) (xpos - lastMouseX) * sensitivity;
            float deltaY = (float) (ypos - lastMouseY) * sensitivity;

            Camera camera = Camera.getInstance();
            camera.setYaw(camera.getYaw() + deltaX);
            camera.setPitch(camera.getPitch() - deltaY);

            // Clamp pitch
            if (camera.getPitch() > 89.0f) camera.setPitch(89.0f);
            if (camera.getPitch() < -89.0f) camera.setPitch(-89.0f);

            lastMouseX = xpos;
            lastMouseY = ypos;
        });

        // Key events
        addKeyPressedEvent(event -> {
            if (event.getKey() == GLFW.GLFW_KEY_ESCAPE) {
                GLFW.glfwSetWindowShouldClose(getWindow(), true);
            }
        });
    }

    @Override
    protected void update(float deltaTime) {
        // WASD movement
        Camera camera = Camera.getInstance();
        float speed = 5.0f * deltaTime;

        // Calculate movement vectors from camera direction
        Vector3f direction = camera.getDirection();
        Vector3f forward = new Vector3f(direction.x, 0, direction.z).normalize();
        Vector3f right = new Vector3f(direction.z, 0, -direction.x).normalize();

        Vector3f movement = new Vector3f();

        if (GLFW.glfwGetKey(getWindow(), GLFW.GLFW_KEY_W) == GLFW.GLFW_PRESS)
            movement.add(forward.mul(speed, new Vector3f()));
        if (GLFW.glfwGetKey(getWindow(), GLFW.GLFW_KEY_S) == GLFW.GLFW_PRESS)
            movement.sub(forward.mul(speed, new Vector3f()));
        if (GLFW.glfwGetKey(getWindow(), GLFW.GLFW_KEY_A) == GLFW.GLFW_PRESS)
            movement.sub(right.mul(speed, new Vector3f()));
        if (GLFW.glfwGetKey(getWindow(), GLFW.GLFW_KEY_D) == GLFW.GLFW_PRESS)
            movement.add(right.mul(speed, new Vector3f()));

        camera.move(movement);
    }

    public static void main(String[] args) throws Exception {
        new MyGame();
    }
}
```

---

## Tips and Best Practices

### Performance

1. **Use quality presets** - Start with MEDIUM and adjust as needed
2. **Limit lights** - Max 8 lights, use fewer for better performance
3. **Bloom resolution** - Use MEDIUM quality for a good balance
4. **Shadow quality** - HIGH (4096) is usually sufficient
5. **Physics substeps** - Use time scaling (e.g., `deltaTime * 0.02f`) for stable physics
6. **MSAA** - 4x provides a good quality/performance balance

### Visual Quality

1. **Enable bloom** for lights and emissive materials
2. **Use PBR materials** for realistic lighting
3. **Set emissive intensity > 1.0** to trigger bloom glow
4. **Adjust exposure** based on scene brightness
5. **Use ACES tone mapping** for a cinematic look

### Physics

1. **Use ground planes** for flat terrain - more efficient than box colliders
2. **Set appropriate restitution** - 0.7 for bouncy balls, 0.1 for heavy objects
3. **Wake up bodies** when applying forces - `body.wakeUp()` before `applyForce()`
4. **Use capsule colliders** for characters - better for stairs/slopes
5. **Scale physics time** - Multiply deltaTime for slow-motion effects
6. **Sync transforms after update** - Physics positions don't update models automatically

### Debugging

1. Enable `setDebugMode(true)` for performance info
2. Use `settings.setWireframeMode(true)` to see geometry
3. Use the built-in `renderImGui()` for real-time tweaking
4. Check `settings.setShowShadowMap(true)` for shadow issues
5. Check `PhysicsWorld.getBodyCount()` and `getContactCount()` for physics stats

---

## API Reference

For detailed API documentation, see the JavaDoc comments in the source code:

### Core Classes
- `io.redstonerdev.gui.window.Window` - Base class for applications
- `io.redstonerdev.threed.render.Renderer3D` - 3D rendering and lighting
- `io.redstonerdev.threed.render.Camera` - View and projection
- `io.redstonerdev.threed.model.Model` - 3D objects with transforms
- `io.redstonerdev.threed.model.Material` - PBR material system
- `io.redstonerdev.threed.model.Shapes` - Procedural mesh generation (cube, sphere, plane, cylinder)
- `io.redstonerdev.threed.lighting.Light` - Lighting system (use factory methods)
- `io.redstonerdev.graphics.font.TextRenderer` - 2D and 3D text
- `io.redstonerdev.threed.render.RenderSettings` - Global settings
- `io.redstonerdev.threed.render.PostProcessing` - Bloom, tone mapping, and MSAA

### Physics Classes
- `io.redstonerdev.threed.physics.PhysicsWorld` - Physics simulation manager (singleton)
- `io.redstonerdev.threed.physics.PhysicsBody` - Rigid body wrapper with JOML interface
- `io.redstonerdev.threed.physics.BodyType` - Enum: STATIC, DYNAMIC, KINEMATIC
- `io.redstonerdev.threed.physics.Collider` - Base interface for collision shapes
- `io.redstonerdev.threed.physics.SphereCollider` - Spherical collision shape
- `io.redstonerdev.threed.physics.BoxCollider` - Oriented bounding box (OBB)
- `io.redstonerdev.threed.physics.CapsuleCollider` - Cylinder with hemispherical caps
- `io.redstonerdev.threed.physics.PlaneCollider` - Infinite plane (static only)
- `io.redstonerdev.threed.physics.AABB` - Axis-aligned bounding box (for broadphase)

### Model Loading
- `io.redstonerdev.threed.loader.GLBLoader` - Load GLB/GLTF models
- `io.redstonerdev.threed.loader.OBJLoader` - Load OBJ/MTL models
