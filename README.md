# Mouse and Keyboard Recorder

A Python-based tool that records mouse movements, clicks, and keyboard inputs, then converts them into reproducible PyAutoGUI scripts.

## Features

- Records mouse movements, clicks (left and right), and keyboard inputs
- Saves recording data in JSON format
- Converts recordings into executable PyAutoGUI scripts
- Supports keyboard shortcuts and mouse buttons
- Includes configurable timing between actions

## Installation

1. Clone this repository
2. Install the required dependencies:
```bash
pip install pynput pyautogui
```

## Usage

### Recording

1. Run the recording script:
```bash
python record.py
```

2. To stop recording:
   - Hold right-click for 2 seconds and release to end mouse recording
   - Press 'ESC' to end keyboard recording
   - Both actions are required to complete the recording

The recording will be saved as `recording.json`.

### Converting to PyAutoGUI Script

1. After recording, convert the JSON file to a PyAutoGUI script:
```bash
python convert.py
```

2. This will generate a `play.py` file that you can run to replay your recorded actions.

### Playing Back

1. Review the generated `play.py` script before running
2. Run the playback script:
```bash
python play.py
```

## How It Works

1. **Recording** (`record.py`):
   - Uses `pynput` to capture mouse and keyboard events
   - Reference: 

```21:43:record.py
recording = [] 
count = 0

def on_press(key):
    try:
        json_object = {
            'action':'pressed_key', 
            'key':key.char, 
            '_time': time.time()
        }
    except AttributeError:
        if key == keyboard.Key.esc:
            print("Keyboard recording ended.")
            return False

        json_object = {
            'action':'pressed_key', 
            'key':str(key), 
            '_time': time.time()
        }
        
    recording.append(json_object)

```


2. **Converting** (`convert.py`):
   - Transforms JSON recording into PyAutoGUI commands
   - Handles key mappings and timing calculations
   - Reference:

```40:94:convert.py
def convert_to_pyautogui_script(recording):
    """
    Converts to a Python template script 'play.py' to 
    use with PyAutoGUI.

    Converts the:

    - Mouse clicks
    - Keyboard input
    - Time between actions calculated
    """
    if not recording: 
        return
    
    output = open("play.py", "w")
    output.write("import time\n")
    output.write("import pyautogui\n\n")
    
    for i, step in enumerate(recording):
        print(step)

        not_first_element = (i - 1) > 0
        if not_first_element:
            ## compare time to previous time for the 'sleep' with a 10% buffer
            pause_in_seconds = (step["_time"] - recording[i - 1]["_time"]) * 1.1 

            output.write(f"time.sleep({pause_in_seconds})\n\n")
        else:
            output.write("time.sleep(1)\n\n")

        if step["action"] == "pressed_key":
            key = step["key"].replace("Key.", "") if "Key." in step["key"] else step["key"]

            if key in key_mappings.keys():
                key = key_mappings[key]

            output.write(f"pyautogui.press('{key}')\n")
        
        if step["action"] == "clicked":
            output.write(f"pyautogui.moveTo({step['x']}, {step['y']})\n")

            if step["button"] == "Button.right":
                output.write("pyautogui.mouseDown(button='right')\n")
            else:
                output.write("pyautogui.mouseDown()\n")

        if step["action"] == "unclicked":
            output.write(f"pyautogui.moveTo({step['x']}, {step['y']})\n")

            if step["button"] == "Button.right":
                output.write("pyautogui.mouseUp(button='right')\n")
            else:
                output.write("pyautogui.mouseUp()\n")
    print("Recording converted. Saved to 'play.py'")
```


## File Structure

- `record.py` - Records mouse and keyboard actions
- `convert.py` - Converts recordings to PyAutoGUI scripts
- `recording.json` - Stores recorded actions
- `play.py` - Generated script for playback

## Important Notes

- Always review the generated `play.py` script before running
- The playback timing might need adjustment based on your system
- Be cautious when running automated mouse/keyboard scripts
- Consider adding error handling for your specific use case

## Limitations

- Scroll events are recorded but simplified in conversion
- Some special keyboard combinations might need manual adjustment
- Timing might vary between systems

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project is open source and available under the MIT License.

---

*Note: This tool is for educational and automation purposes. Use responsibly and always review generated scripts before running them.*