
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