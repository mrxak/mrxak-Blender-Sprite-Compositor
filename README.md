# mrxak's Blender Sprite Compositor
This .blend file has a complete compositing pipeline for generating sprites for EV Nova and Cosmic Frontier. Model your ships in this .blend file (or import them) and you will be able to get all the necessary sprites and masks with a single render pass.

You get:
* /Frames/CoFro_Ship/ShipSpriteXXXX.png - A base ship sprite with alpha transparency
* /Frames/Nova_Ship/ShipSpriteXXXX.png - A base ship sprite with a black background and no transparency
* /Frames/Nova_ShipMasks/ShipMaskXXXX.png - A black and white mask for use with your non-transparent base ship sprite
* /Frames/CoFro_Engine/EngineSpriteXXXX.png - An engine glow sprite with alpha transparency
* /Frames/Nova_Engine/EngineSpriteXXXX.png - A engine glow sprite with a black background and no transparency
* /Frames/Nova_EngineMasks/EngineMaskXXXX.png - A black and white mask for use with your non-transparent engine glow sprite
* /Frames/CoFro_Lights/LightsSpriteXXXX.png - A running lights sprite with alpha transparency
* /Frames/Nova_Lights/LightsSpriteXXXX.png - A running lights sprite with a black background and no transparency
* /Frames/Nova_LightsMasks/LightsMaskXXXX.png - A black and white mask for use with your non-transparent running lights sprite
* /Frames/Composite/CompositeXXXX.png - A composite of all three layers with black background and no transparency

You will probably want to use either the CoFro outputs or the Nova outputs, not both, unless you are make a plug-in for both games simultaneously. The composite output is useful for making a spinning GIF of your ship as it will look in game with everything turned on.

A lot of magic happens in the Compositing nodes, but they work thanks to render layers in the .blend file. These layers are set up for you with proper activations/deactivations and holdouts, and you shouldn't need to do anything with them to get all the necessary render passes.

The layers are:
* 01.Ship Render - The ship is rendered with sunlight and starlight, but nothing else
* 02.Engine Glow Render - The ship is rendered in total darkness, lit only by the ship's engine glow
* 03.Running Lights Render - The ship is rendered in total darkness, lit only by the ship's running lights.
* MODEL EVERYTHING HERE - This layer is not rendered, but it's recommended you do all of your ship's modeling and animation set-up in this layer.

![Render Layers](https://user-images.githubusercontent.com/5839156/148002546-c8fc2fcc-7055-48e3-9614-66cd82f66694.png)

In order to let the render layers work correctly, you must place the components of your ship into the right collections. All engine glows need to be placed in the Engine Glows collection. All running light emissives need to be placed in the Running Lights collection. The rest of your ship needs to be placed in the Ship collection. You can still have all of your components parented to your main ship object, to be animated over a number of frames. You can use as many frames as you like, but I recommend 36 for very small ships, 64 for small and medium ships, and then 100 or 144 frames for large and very large ships so they spin smoothly and don't look so obviously like sprites. You may need to multiply those values if you have more complicated sprite set-ups like banking frames and per-sprite animations. The .blend file comes with an example ship that is already animated to spin clockwise over 64 frames, and set up correctly in the collections. Note that the first frame is set to a rotation of 0 degrees on the Z axis and the end key frame is one frame longer than the animation at -360 degrees on the Z axis. The rotation animation is set to linear interpolation mode. This is how you get a nice spin sprites.

![Sprite Collections](https://user-images.githubusercontent.com/5839156/148003390-7573e111-3549-4bdb-84c1-035a3ad41e5e.png)

You can adjust the sun and camera if needed, but by default there are some sensible settings. The default sun is set to a rotation of 65 degrees on the X axis and a rotation of 40 degrees on the Z axis. The sun's location is driven by the camera location in order to keep it out of the way of your ship, however big it might be. The default camera is set to a rotation of 45 degrees on the X axis and 180 degrees on the Z axis. It's highly recommended to keep the camera at 0 on the X axis, and only adjust the Y axis as needed. The camera will automatically adjust itself on the Z axis based on your Y translation. Simply drag your camera's Y location and the camera will stay pointed at the origin.

The 45 degree camera is for use in EV Nova, which at the time of this project was still the dominant target for new plug-in graphics. If you'd like to set up your camera for Cosmic Frontier, you'll want to set it at a Y of 0 and then adjust your Z to whatever height you like. Just delete the Z driver by right-clicking on the Z field and choose Delete Drivers. Then change the camera's X rotation to 0.

The compositing nodes will halve the size of your renders when they output their files. It's recommended you use square render dimensions that are a multiple of 8 pixels wide, and 8 pixels tall, and twice the size you want your final ship sprites to be. This allows for the optional use of the "LASIK Method" first introduced in EV Nova, and provides your ships with a bit of antialiasing regardless.

## Changing the Compositor Nodes

![Compositor Node Overview](https://user-images.githubusercontent.com/5839156/148001289-8de70bf6-b656-4198-83da-8ad13a6ab306.png)

I imagine you'll only want to adjust certain things on the compositor nodes, and I'll highlight those here. Generally, if a node is minimized, it's something you shouldn't mess with unless you really know what you're doing.

As you can see there are several groups of nodes, in labeled boxes. These read left-to-right, and terminate in specific outputs, which are labeled.

![Ship Group](https://user-images.githubusercontent.com/5839156/148005253-97e7649f-0991-444a-86e3-d6b12e2e44a6.png)

The Ship group takes the rendered image each frame of your animation and its alpha values, and directs them through different sub-groups. First is the LASIK Method nodes, which, if the Switch node switch is checked, applies a sharpening to your rendered ship. Then, whether you applied sharpening or not, it scales the image down. At the same time, a black and white mask is created. Finally, some optional extra cropping is done to your images, and you get three outputs: two sets of sprites (one with transparency, one without), and then another set of mask for use with the sprites without transparency. The optional extra cropping is explained below.

You may wish to change the output file locations, but by default a file hierarchy will be created in the same directory as wherever you have this .blend file saved. Disconnect the "noodles" or wires that go into any file output nodes you don't wish to output.

![Engine Glow Group](https://user-images.githubusercontent.com/5839156/148005788-81eaa836-a6d5-4272-be1e-5e9156460737.png)

The Engine Glow Group takes the rendered image each frame of your animation, its alpha values, and its emitter values. If you've set up your collections correctly, your ship glows will be in the emitter values. These then get processed through different sub-groups. First the compositor nodes isolate the reflections (and direct light) from your engine glows and keeps only those pixels. At the same time, the compositor identifies the pixels that are from emitters, and applies a bloom effect using a Glare node. You may wish to adjust the size of the bloom, or the glow falloff value, which constraints how far from the emitter pixels the bloom can get. The bloom effect gets added to the image, and its alpha. The rest of the nodes in this group are of a familiar structure which do the same things as in the Ship group, just without any LASIK step. The optional extra cropping is explained below.

You may wish to change the output file locations, but by default a file hierarchy will be created in the same directory as wherever you have this .blend file saved. Disconnect the "noodles" or wires that go into any file output nodes you don't wish to output.

![Running Lights Group](https://user-images.githubusercontent.com/5839156/148006566-340b7791-5512-4846-b614-d39c5fcc5fa8.png)

The Running Lights Group works identically to the Engine Glow group above, just with the running lights instead of the engine glow. Here too is available settings for size of bloom, and glow falloff. The optional extra cropping is explained below.

You may wish to change the output file locations, but by default a file hierarchy will be created in the same directory as wherever you have this .blend file saved. Disconnect the "noodles" or wires that go into any file output nodes you don't wish to output.

![Composite Group](https://user-images.githubusercontent.com/5839156/148006781-f28b4436-b279-4be1-9555-5201ce224535.png)

The Composite group simply assembles the three component layers into a single output file, as well as showing the composite in your image viewer while rendering, and a separate viewer on the Compositing tab itself. You may wish to change the output file location, but by default a file hierarchy will be created in the same directory as wherever you have this .blend file saved. If you don't need these fully composited frames, disconect the "noodle" or wire that goes into the file output node.

![extracroppingdrivers](https://user-images.githubusercontent.com/5839156/148007085-bcf37ca4-52cf-4f1a-a87b-4e2910eee338.png)

Finally, there is the Extra Cropping Drivers group. These nodes appear to not be connected to anything, but they are in fact drivers, used by the bright green nodes elsewhere in the overall node tree. These three nodes, labeled Ship, Engine, and Lights, give you extra control over the final output files. Adjusting these values by whole integer numbers will crop that number of pixels from around each frame in the respective output files. As you can see with the example ship, the engine glows extend quite a bit further out from the world origin, and the running lights are more centralized than the ship itself. You can trim away any wasted space by cropping with these nodes, keeping your file sizes down.
