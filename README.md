[![Static Badge](https://img.shields.io/badge/Godot%20Engine-4.5.rc2-blue?style=plastic&logo=godotengine)](https://godotengine.org/)
# Dragonforge Camera
A camera node to allow easy switching between multiple camera angles and modes.
# Version 0.2
For use with **Godot 4.5.rc2** and later.
## Dependencies
The following dependencies are included in the addons folder and are required for the template to function.
- [Dragonforge Controller 0.12](https://github.com/dragonforge-dev/dragonforge-controller)
# Installation Instructions
1. Copy the `dragonforge_controller` folder from the `addons` folder into your project's `addons` folder.
2. Ignore the following errors (they are appearing because the component is not yet enabled):
  * ERROR: res://addons/dragonforge_controller/controller.gd:54 - Parse Error: Identifier "Keyboard" not declared in the current scope.
  * ERROR: res://addons/dragonforge_controller/controller.gd:56 - Parse Error: Identifier "Mouse" not declared in the current scope.
  * ERROR: res://addons/dragonforge_controller/controller.gd:59 - Parse Error: Identifier "Gamepad" not declared in the current scope.
  * ERROR: modules/gdscript/gdscript.cpp:3022 - Failed to load script "res://addons/dragonforge_controller/controller.gd" with error "Parse error".
3. In your project go to **Project -> Project Settings...**
4. Select the **Plugins** tab.
5. Check the **On checkbox** under **Enabled** for **Dragonforge Controller**
6. Press the **Close** button. (If you would like to ensure the errors are gone, go to **Project -> Reload Project**. When the project reloads, the previous errors should no longer appear. (We cannot guarantee your own errors will not still appear.))
7. In your project go to **Project -> Project Settings...**
8. Select the **Input Map** tab.
9. Add a new action named `change_camera`.
10. Map the action **Joypad Button 8** (right-stick press) and the **C** key. (Or whatever you choose)
11. Copy the `dragonforge_camera` folder from the `addons` folder into your project's `addons` folder.

# Usage Instructions
1. On your **CharacterBody3D** node click **+ Add Child Node...** and select a **Cameras** node.
2. On your **Cameras** node click **+ Add Child Node...** and add as many **Camera3D** nodes as you like. Configure them however you like. The player will be able to rotate through them at will.
## First-Person View Camera
1. If you haven't already, on your **CharacterBody3D** node click **+ Add Child Node...** and select a **Cameras** node.
2. On your **Cameras** node click **+ Add Child Node...** and add a **CameraMount3D** node.
3. Change the **Upwards Rotation Limit** to **-15**.
4. Change the **Downwards Rotation Limit** to **40**.
5. Check the **First Person** box.
6. Add the following code to your **CharacterBody3D**:

```
class_name Player extends Character


## Your character model
@export var rig: Node3D
## The speed at which the player turns.
@export var animation_decay: float = 20.0
##
@export var speed = 5.0


#A reference to your Cameras object
@onready var cameras: Cameras = $Cameras


var direction := Vector3.ZERO


func _physics_process(delta: float) -> void:
	direction = get_input_direction()
	
	velocity.x = direction.x * speed
	velocity.z = direction.z * speed
	
	if velocity.length() > 1.0 and direction != Vector3.ZERO:
		look_toward_direction(delta)
	
	move_and_slide()
	

func get_input_direction() -> Vector3:
	var camera = cameras.active_camera
	var input_dir := Input.get_vector("move_left", "move_right", "move_forward", "move_backward")
	var input_vector := Vector3(input_dir.x, 0, input_dir.y).normalized()
	if camera is CameraMount3D:
		return camera.horizontal_pivot.global_transform.basis * input_vector
	elif camera.rotation.y != 0.0:
		return input_vector.rotated(Vector3.UP, camera.rotation.y).normalized()
	else:
		return transform.basis * input_vector


func look_toward_direction(delta: float) -> void:
	var target := rig.global_transform.looking_at(rig.global_position + direction, Vector3.UP)
	rig.global_transform = rig.global_transform.interpolate_with(target, 1.0 - exp(-animation_decay * delta))
```

# Class Descriptions
## Cameras
A list of cameras. If you want a first-person or third-person camera, attach a **CameraMount3D** node as a child of this one (you can add as many as you want.) If you want to add a static camera, you can do that too.
### OnReady Variables
- `available_cameras: Array[Node3D]` The cameras available to the player. Pressing the change_camera button will switch to the next camera in the list. The list is constructed when this object is first created, and is made of all the child nodes one level down that are either **Camera3D** of **CameraMount3D** nodes.
- `active_camera_iterator` Iterator
- `active_camera: Node3D = get_first_camera()` A reference to the currently active camera. This currently ONLY tracks cameras the player has control over (by switching). Any cutscene cameras, etc. will not be assigned to this variable.
### Public Functions
Note that while all these functions are available, as long as the `change_camera` action is mapped, the played can cycle through all the available cameras without any additional coding. These functions are primarily for switching between player cameras and cutscene cameras.
- `next_camera() -> void` Activates the next camera in the list.
- `get_first_camera() -> Node3D` Return the first Camera3D or CameraMount3D found that is a child of this node.
- `inititalize_cameras() -> Array[Node3D]` Return a list of all Camera3D and CameraMount3D nodes that are children of this node.
- `change_camera(camera: Node3D) -> void` Makes the passed camera the active camera.

## CameraMount3D
Intended if you want a first person or third person camera attached to the player. If you want this to be a first person camera, set `first_person` to true.
### Export Variables
- `upwards_rotation_limit: float = 0.0` How far up the camera will rotate in degrees.
- `downwards_rotation_limit: float = 0.0` How far down the camera will rotate in degrees.
- `first_person: bool = false` If true, camera is a first-person camera. Otherwise, third-person.
### Public Functions
- `make_current() -> void` Make the attached camera the current camera and reset the rotation.
