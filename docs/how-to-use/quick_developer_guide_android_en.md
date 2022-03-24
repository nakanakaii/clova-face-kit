# Quick Developer Guide to CLOVA Face Kit for Android

## Exampele Project

Please refer to CLOVA Face Kit application method and usage [examples/android](../../examples/android).

## How to use

1. Create a `ClovaSee` object with a `Context`. When creating a `ClovaSee` object, you can set some detailed options through `Options`, which are explained in the next section.
   ```Kotlin
   import ai.clova.see.ClovaSee

   val clovaSee = ClovaSee(context)
   ```

2. Prepare the image to be analyzed with ‘ClovaSee’ by making it ‘Bitmap’.

   ```Kotlin
   val bitmap = getBitmapFromSomewhere(...)
   ```

3. Set `clova::face::Options` using `face.OptionsBuilder()` to determine which information to analyze from the face in `Bitmap` and to set conditions. Options that can be set in `face.Options` are described in the last section.

   ```Kotlin
   val faceOptions = ai.clova.see.face.OptionsBuilder()
                               .setBoundingBoxThreshold(0.7f)
                               .setInformationToObtain(ai.clova.see.face.Options.CONTOURS or
                                                       ai.clova.see.face.Options.MASKS or
                                                       ai.clova.see.face.Options.EULER_ANGLES or
                                                       ai.clova.see.face.Options.TRACKING_IDS)
                               .setResizeThreshold(320)
                               .setMinimumBoundingBoxSize(0.1f)
                               .build()
   
   ```

4. Call `ClovaSee.run()` with `Bitmap` prepared in step 2 and `face.Options` prepared in step 3. `ClovaSee.run()` analyzes the face information in the given `Bitmap` and returns the result in `face.Result`. The information returned to `Face` is explained in the last section.

   ```Kotlin
   import ai.clova.see.Face

   val faces = clovaSee.run(bitmap, faceOptions.build())
   ```

5. Do whatever you want with the `Array<Face>` returned in 3.

   ```Kotlin
   import ai.clova.see.Contour
   import android.graphics.Canvas
   import android.graphics.Paint

   faces.forEach { face ->
       val canvas = getCanvasFromSomewhere(...)
       val paint = Paint()

       // Draw the face bounding box.
       canvas.drawRect(face.boundingBox, paint)
       // Draw the face contour points.
       face.contour.points.forEach { canvas.drawPoint(it.x.toFloat(), it.y.toFloat(), paint) }

       ...
   }
   ```

## How to set detailed options

By passing a `Settings` object as the second parameter when creating a `ClovaSee` object, you can control several detailed options regarding how `ClovaSee` behaves. A `Settings` object is usually created through `SettingsBuilder`. Examples of usage and the meaning of main options are as follows.

```Kotlin
import ai.clova.see.ClovaSee
import ai.clova.see.Settings
import ai.clova.see.SettingsBuilder

val settings = SettingsBuilder()
    .setNumberOfThreads(4)
    .setPerformanceMode(Options.PerformanceMode.ACCURATE_98)
    .setIntermittentInformationRatio(1)
    .build()
val clovaSee = ClovaSee(context, settings)
```

1. `SettingsBuilder.setNumberOfThreads()`: set the size of the thread pool inside `ClovaSee`. This is usually set to the number of cores installed in the execution environment (the number of Bigs in the case of Big-Little architectures), the default value is 4.

2.`SettingsBuilder.setPerformanceMode()`: Sets which is more important, the speed of `ClovaSee` or the accuracy of the information returned. You can set one of the values below, and the default value is `Settings.PerformanceMode.ACCURATE_98`. Currently, the only return value affected by this option is `Face.contours`, but this may be expanded in the future.

   `Settings.PerformanceMode.ACCURATE_106`: Return 106 point contour information in `Face.contours`.

   `Settings.PerformanceMode.ACCURATE_98`: Returns 98-point contour information in `Face.contours`.

   `Settings.PerformanceMode.FAST`: Returns contour information made up of 5 points in `Face.contours`.

3. `SettingsBuilder.setIntermittentInformationRatio()`: Specifies the interval to perform face analysis in number of frames. Default is 1. For example, if 5 is specified, the information set with `OptionsBuilder.setInformationToObtain()` is analyzed and returned only once every 5 frames. However, `Options.BOUNDING_BOXES` is always returned regardless of this cycle. In this way, information that is analyzed and returned only according to the cycle set by the user is called Intermittent Information, and all information except `Options.BOUNDING_BOXES` corresponds to this.


## `face.Options`

1. `OptionsBuilder.setInformationToObtain()`: Set the type of information you want to know through `ClovaSee`. You can set the type of information you want among the values below by grouping it with ‘or’. Simply setting only the kind of information you need will help reduce the execution time of `ClovaSee.run()`. The default is `Options.ALL` .

   `Options.BOUNDING_BOXES`: Returns the coordinate information of the area where the face is located.

   `Options.CONTOURS`: Returns the outline information of the face.

   `Options.MASKS`: Returns whether a mask is worn.

   `Options.TRACKING_IDS`: Returns an ID assigned to each face.

   `Options.MOJOS`: Returns a face information value called `Mojo`. Information used only by some services.
   
   `Options.SPOOFS`: Returns whether the face is real or fake (device, print, mask).

   `Options.FEATURES`: Returns the face feature values.

   `Options.EULER_ANGLES`: Returns the direction the face is facing as X, Y, Z values.

   `Options.ALL`: Returns all six pieces of information above.


## Information returned to `Face`

`ClovaSee.run()` analyzes the face in the passed `Bitmap` and returns the result in `Array<Face>`. The information returned to `Face` is as follows:

1. `Face.boundingBox`: The coordinate information of the area with the face is returned in `Rect` format. Information corresponding to the green square area in Figure 1 below.

2. `Face.contour`: The contour information of the face is returned in `Contour` format. Information corresponding to the points drawn along the outline of the face in Figure 1 below.

   `Options.PerformanceMode.ACCURATE_106`: returns contour information consisting of 106 points. The location and index information of the points is shown in Figure 2.

   `Options.PerformanceMode.ACCURATE_98`: returns an outline consisting of 98 points. The location and index information of the points is shown in Figure 3.

   `Options.PerformanceMode.FAST`: returns contour information consisting of 5 points. The location of the point corresponds to the red dot in Figure 4.

3. `Face.mojo`: Information used only by some services.

4. `Face.feature`: The face feature value is returned in `Feature` format.

5. `Face.eulerAngle`: The direction the face is facing is returned in the format `EulerAngle`.

   `EulerAngle.x()`, `EulerAngle.pitch()`: The angle when you move your head up and down while looking straight ahead. Values range from -90 to 90 degrees, and moving down returns a negative number.

   `EulerAngle.y()`, `EulerAngle.yaw()`: The angle when the head is moved left and right while looking straight ahead. Values range from -90 to 90 degrees, and moving left returns a negative number.

   `EulerAngle.z()`, `EulerAngle.roll()`: This is the angle when you tilt your head toward the left and right shoulders while looking straight ahead. Values range from -90 to 90 degrees, and moving towards the left shoulder returns a negative number.

6. `Face.trackingID`: A unique ID assigned to each face is returned in the format `TrackingID`. This ID is zero-based and is always positive.

7. `Face.mask`: Whether to wear a mask is returned in `Mask` format. This format is equivalent to `Boolean`. `true` means wearing a mask, `false` means not wearing a mask.

<img src="../figure_1.png" width=50%/><br/>
*Picture 1*

<img src="../figure_2.png" width=50%/><br/>
*Picture 2. 106 Points*

<img src="../figure_3.png" width=50%/><br/>
*Picture 3. 98 Points*

<img src="../figure_4.png" width=50%/><br/>
*Picture 4*
