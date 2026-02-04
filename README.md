# RedEngine

A modern, feature-rich 3D rendering engine for Java built on LWJGL 3 and OpenGL.

## Features

- **PBR Rendering** - Physically Based Rendering with metallic-roughness workflow
- **Dynamic Lighting** - Directional, point, and spot lights with shadow mapping
- **Post-Processing** - HDR, bloom, tone mapping (Reinhard, ACES, Uncharted2)
- **MSAA Anti-Aliasing** - Configurable 2x, 4x, 8x, 16x multisampling
- **Physics System** - Rigid body dynamics powered by ODE4J
- **Text Rendering** - 2D screen-space and 3D billboard text with SDF support
- **Model Loading** - GLB/GLTF with skeletal animations, OBJ/MTL support
- **Material System** - Pre-built metals, plastics, glass, and emissive materials
- **ImGui Integration** - Built-in debug UI and settings panels
- **Cross-Platform** - Windows, Linux, and macOS support

## Requirements

- Java 17 or higher
- OpenGL 3.3+ compatible GPU

## Installation

### Maven

**Platform-specific JAR with bundled natives (Recommended)**

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

**Core library only (provide your own natives)**

```xml
<dependency>
    <groupId>io.redstonerdev</groupId>
    <artifactId>RedEngine</artifactId>
    <version>0.6.0</version>
</dependency>
```

## Quick Start

```java
import io.redstonerdev.graphics.RenderI;
import io.redstonerdev.graphics.font.TextRenderer;
import io.redstonerdev.graphics.render.Renderer;
import io.redstonerdev.gui.window.Window;
import io.redstonerdev.threed.lighting.Light;
import io.redstonerdev.threed.model.Material;
import io.redstonerdev.threed.model.Model;
import io.redstonerdev.threed.model.Shapes;
import io.redstonerdev.threed.render.Camera;
import io.redstonerdev.threed.render.Renderer3D;
import org.joml.Vector3f;

import java.awt.Color;
import java.util.ArrayList;
import java.util.List;

public class MyGame extends Window {

    private List<Model> models = new ArrayList<>();

    public MyGame() throws Exception {
        super(0, 0, 1280, 720);
        setTitle("My Game");

        // Create a simple scene using Shapes class
        Model ground = new Model(
            Shapes.createPlaneMesh(20),
            Material.createPlastic(new Vector3f(0.3f), 0.8f)
        );
        models.add(ground);

        Model cube = new Model(
            Shapes.createCubeMesh(),
            Material.createGold()
        );
        cube.setPosition(new Vector3f(0, 0.5f, -5));
        models.add(cube);

        // Add lighting using factory methods
        Light sun = Light.createDirectional(
            new Vector3f(1, -2, 1),
            new Vector3f(1, 1, 1),
            1.0f
        );
        Renderer3D.getInstance().addLight(sun);

        // Setup camera
        Camera.getInstance().setPosition(new Vector3f(0, 2, 5));

        // Add renderer
        addRenderer((renderer, renderer3D, textRenderer) -> {
            renderer3D.renderModels(models);
            textRenderer.renderText(
                getFps() + " FPS", 10, 10, 24, Color.WHITE,
                renderer.getWidth(), renderer.getHeight()
            );
        });

        loop();
    }

    public static void main(String[] args) throws Exception {
        new MyGame();
    }
}
```

## Core Components

| Component | Description |
|-----------|-------------|
| `Window` | Base application class with game loop |
| `Renderer3D` | 3D rendering, lighting, and shadows |
| `Camera` | View and projection management |
| `Model` | 3D objects with transforms and materials |
| `Shapes` | Procedural mesh generation (cube, sphere, plane, cylinder) |
| `Material` | PBR material system |
| `Light` | Directional, point, and spot lights (use factory methods) |
| `TextRenderer` | 2D and 3D text rendering |
| `PhysicsWorld` | Rigid body physics simulation |
| `RenderSettings` | Global quality and rendering options |
| `PostProcessing` | Bloom, tone mapping, MSAA |

## Material Examples

```java
// Metals
Material gold = Material.createGold();
Material steel = Material.createBrushedSteel();

// Custom metallic
Material chrome = Material.createMetallic(
    new Vector3f(0.9f), 1.0f, 0.1f
);

// Emissive (glowing)
Material neon = Material.createEmissive(
    new Vector3f(1, 0, 0), 5.0f  // intensity > 1 triggers bloom
);

// Glass
Material glass = Material.createGlass(
    new Vector3f(1, 1, 1), 0.2f
);
```

## Physics Example

```java
import io.redstonerdev.threed.physics.*;

PhysicsWorld world = PhysicsWorld.getInstance();
world.setGravity(new Vector3f(0, -9.81f, 0));

// Create dynamic sphere
SphereCollider shape = new SphereCollider(0.5f);
PhysicsBody ball = world.createDynamicBody(shape, 1.0f);
ball.setPosition(new Vector3f(0, 10, 0));
ball.setRestitution(0.7f);

// Create static ground
PhysicsBody ground = world.createGroundPlane(0);

// In update loop
world.update(deltaTime * 0.02f);
model.setPosition(ball.getPosition());
model.setRotation(ball.getRotationEuler());
```

## Render Settings

```java
RenderSettings settings = RenderSettings.getInstance();

// Apply quality preset
settings.applyPreset(RenderSettings.QualityPreset.HIGH);

// Enable effects
settings.setBloomEnabled(true);
settings.setBloomIntensity(1.0f);
settings.setToneMapping(true);
settings.setToneMappingOperator(RenderSettings.ToneMappingOperator.ACES);

// MSAA
settings.setAntiAliasing(RenderSettings.AntiAliasing.MSAA_4X);
```

## Documentation

For comprehensive documentation including:
- Detailed API usage
- Lighting and shadow configuration
- Camera controls and input handling
- Physics system guide
- Post-processing effects
- ImGui integration
- Model loading and animations

See the [Engine Guide](ENGINE_GUIDE.md).

## Dependencies

- [LWJGL 3](https://www.lwjgl.org/) - OpenGL, GLFW, STB, OpenAL bindings
- [JOML](https://github.com/JOML-CI/JOML) - Java OpenGL Math Library
- [ImGui-Java](https://github.com/SpaiR/imgui-java) - Immediate mode GUI
- [ODE4J](https://github.com/tzaeschke/ode4j) - Open Dynamics Engine for Java
- [jglTF](https://github.com/javagl/JglTF) - GLTF model loading

