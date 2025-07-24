
# Unity 2022 Learning
 - by Dawid Jakubowski

## Free Camera Looking


### Example Code

> Use the camera available in the default 3D project, and attach this script (Camera.cs) to this camera.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(Camera))]
public class Camera : MonoBehaviour
{
    private Camera cam;

    [Header("Movement")]
    public float moveSpeed = 10f;
    public float rotationSpeed = 100f;
    public float zoomSpeed = 5f;

    [Header("Cliping Planes")]
    public float nearClip = 0.3f;
    public float farClip = 1000f;

    [Header("Background")]
    public CameraClearFlags clearFlags = CameraClearFlags.Skybox;
    public Color backgroundColor      = Color.black;

    [Header("Projection")]
    public bool startOrthographic = false;
    public float orthographicSize = 5f;     // for 2D
    public float minFOV           = 15f;    // for 3D
    public float maxFOV           = 90f;

    [Header("Culling")]
    public LayerMask cullingMask = ~0;      // all layers by default

    void Awake()
    {
        cam = GetComponent<Camera>();

        cam.nearClip = nearClip;
        cam.farClip = farClip;
        cam.clearFlags = clearFlags;
        cam.backgroundColor = backgroundColor;
        cam.cullingMask = cullingMask;
        cam.startOrthographic = startOrthographic;

        if (startOrthographic)
        {
            cam.orthographicSize = orthographicSize;
        }
    }

    void Update()
    {
        HandleMovement();    // WASD/Arrows
        HandleRotation();    // Mouse X/Y
        HandleZoom();        // Scroll wheel
        HandleProjectionToggle(); // C key
    }
    void HandleMovement()
    {
        // Input.GetAxis("Horizontal") covers A/D and Left/Right
        // Input.GetAxis("Vertical")   covers W/S and Up/Down
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");

        // Move in local XZ plane
        Vector3 delta = (transform.right * h + transform.forward * v)
                        * moveSpeed * Time.deltaTime;
        transform.position += delta;
    }
    void HandleRotation()
    {
        // Right‑mouse held down? (optional gate)
        if (Input.GetMouseButton(1))
        {
            float mX = Input.GetAxis("Mouse X") * rotationSpeed * Time.deltaTime;
            float mY = Input.GetAxis("Mouse Y") * rotationSpeed * Time.deltaTime;

            // Yaw (around world‑up) and pitch (around local‑right)
            transform.Rotate(Vector3.up, mX, Space.World);
            transform.Rotate(Vector3.right, -mY, Space.Self);
        }
    }

    void HandleZoom()
    {
        float scroll = Input.GetAxis("Mouse ScrollWheel");
        if (Mathf.Abs(scroll) < 0.01f) return;

        if (cam.startOrthographic)
        {
            cam.orthographicSize = Mathf.Max(0.1f,
                cam.orthographicSize - scroll * zoomSpeed);
        }
        else
        {
            cam.minFOV = minFOV;
            cam.maxFOV = maxFOV;
        }
    }

    void HandleProjectionToggle()
    {
        if (Input.GetKeyDown(KeyCode.C))
        {
            cam.startOrthographic = !cam.startOrthographic;

            // If switching into ortho, set its size
            if (cam.startOrthographic)
                cam.orthographicSize = orthographicSize;
        }
    }

    // Start is called before the first frame update
    void Start()
    {

    }


}
```

### Explaination:

------------------------------------

```csharp
private Camera cam;
```
 - The reference of the camera, 
 - How: Fetched with GetComponent<Camera>() in Awake()

------------------------------------

```csharp
public float moveSpeed = 10f;
```

 - Controls the translation speed when pressing WASD or Arrow Keys
 - Affects transform.position via Input.GetAxis(...)
 - Default: 10 Unity units/second

------------------------------------

```csharp
public float rotationSpeed = 100f;
```

 - Controls mouse look sensitivity
 - Affects transform.Rotate(...) in world and local space
 - 100 is fast but usable; tweak to fit your feel

------------------------------------

```csharp
public float zoomSpeed = 5f;
```

 - Controls how fast zoom reacts to scroll wheel
 - Affects either:
     -Camera.fieldOfView in Perspective mode
     -Camera.orthographicSize in Orthographic mode

------------------------------------

```csharp
public float minFOV = 15f;
public float maxFOV = 90f;
```

 - Min/max constraints for 3D zoom (Perspective)
 - Prevents extreme lens distortion or invisibility

------------------------------------

```csharp
public float orthographicSize = 5f;
```

 - Camera size (height from center to top) in 2D (Orthographic) view
 - Smaller value → zoomed in
 - Only used if cam.orthographic == true

------------------------------------

```csharp
public bool startOrthographic = false;
```

 - Toggles whether camera starts in 2D (orthographic) or 3D (perspective) at runtime
 - Used during initialization in Awake()

------------------------------------

```csharp
public float nearClip = 0.3f;
public float farClip = 1000f;
```

 - Near/far bounds of what camera renders:
     - Anything closer than nearClipPlane or farther than farClipPlane is invisible
     - Default Unity camera uses 0.3 and 1000 respectively

------------------------------------

```csharp
public CameraClearFlags clearFlags = CameraClearFlags.Skybox;
```

 - Determines what the camera renders behind your objects:
     - Skybox: default
     - SolidColor: flat color (e.g., black)
     - DepthOnly: only affects depth buffer (for stacked cameras)
     - Nothing: renders nothing

------------------------------------

```csharp
public Color backgroundColor = Color.black;
```
 - If clearFlags = SolidColor, this defines that color
 - Color.black, Color.red, etc. are Unity constants

------------------------------------

```csharp
public LayerMask cullingMask = ~0;
```

 - Tells the camera which object layers to render
 - ~0 means “all layers”
 - Used for rendering only specific types of objects (e.g., HUD layer, enemies, etc.)

------------------------------------

```csharp
Input.GetAxis("Horizontal") // A/D or Left/Right
Input.GetAxis("Vertical")   // W/S or Up/Down
Input.GetAxis("Mouse X")    // Horizontal mouse movement
Input.GetAxis("Mouse Y")    // Vertical mouse movement
Input.GetAxis("Mouse ScrollWheel") // Mouse wheel
Input.GetMouseButton(1)     // Right mouse button held
Input.GetKeyDown(KeyCode.C) // Toggle projection key
```

 - These values are float inputs (mostly -1 to 1), mapped by Unity’s Input Manager (Project Settings → Input).

| Variable            | Type             | Controls            | Applies To                          |
|---------------------|------------------|---------------------|-------------------------------------|
| `cam`               | Camera Component | access              | Camera on GameObject                |
| `moveSpeed`         | float            | Camera panning (XZ) | `transform.position`                |
| `rotationSpeed`     | float            | Mouse look          | `transform.Rotate`                  |
| `zoomSpeed`         | float            | Scroll zoom rate    | `fieldOfView` or `orthographicSize` |
| `minFOV, maxFOV`    | float            | Zoom limits         | Perspective only                    |
| `orthographicSize`  | float            | 2D zoom             | Orthographic only                   |
| `startOrthographic` | bool             | Start in 2D or 3D   | Initialization                      |
| `nearClip, farClip` | float            | Render depth        | Camera clipping                     |
| `clearFlags`        | CameraClearFlags | BG render mode      | Clear method                        |
| `backgroundColor`   | Color            | BG color (flat)     | If SolidColor                       |
| `cullingMask`       | LayerMask        | What to render      | Object layers                       |
