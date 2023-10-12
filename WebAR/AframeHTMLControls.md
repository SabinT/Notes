# Interactive A-Frame Slider Example

Example of creating an HTML slider to control the x-position of an A-Frame entity.

```html
<!DOCTYPE html>
<html>
  <body style="margin: 0; padding: 0; overflow: hidden; position: relative;">
    <a-scene style="position: absolute; width: 100%; height: 100%;">
      <a-box id="myBox" position="0 1 5" rotation="0 0 0" color="#4CC3D9"></a-box>
    </a-scene>

    <div style="position: absolute; top: 20px; left: 20px; z-index: 1;">
      <input type="range" id="slider" min="0" max="10" value="0" step="0.1" />
    </div>

    <script>
      document.addEventListener('DOMContentLoaded', function () {
        const slider = document.getElementById('slider');
        const box = document.getElementById('myBox');
        
        slider.addEventListener('input', function () {
          const x = slider.value;
          box.setAttribute('position', { x: x, y: 1, z: 5 });
        });
      });
    </script>
  </body>
</html>
```

## Explanation

- The `a-scene` contains an `a-box` A-Frame entity, positioned absolutely to cover the entire viewport.
- The `input` element of type `range` acts as the slider. It is also positioned absolutely but has a `z-index` of 1 to ensure it appears above the A-Frame scene. The `step` attribute is set to 0.1, allowing for smoother transitions.
- The JavaScript listens for the `DOMContentLoaded` event to ensure the elements exist before attaching the event listener. It then listens for the `input` event on the slider. When the slider is adjusted, it updates the x-position of the `a-box` entity.
