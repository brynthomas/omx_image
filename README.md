omx_image
=========

An example image loader for OpenMAX on the Raspberry PI

As someone new to the OpenMAX usage club I very quickly wished "can't someone just give a simple example that will decode a JPEG into an OpenGL ES texture that I can use in my code".

So I wrote a bit of code which is for use in another app I'm doing, but here's some instructions on how to integrate it into an existing app.

**How to make the hello_triangle example read a texture from a JPG with OpenMAX**


* Duplicate the hello_triangle directory into a hello_triangle_jpeg directory, but leave it in the same hello_pi base folder since it needs to use the shared Makefile. (the hello_triangle example being the one in https://github.com/raspberrypi/firmware/tree/master/opt/vc/src/hello_pi )

* Download the omx_event_and_buffer_handler.c, omx_event_and_buffer_handler.h, omx_image.c and omx_image.h files from this repository into the hello_triangle_jpeg directory. As a sample image also download the Raspi_Colour_R.jpg into that directory.

* Open the Makefile and change the line:

```
OBJS=triangle.o
```

to

```
OBJS=omx_event_and_buffer_handler.o omx_image.o triangle.o
```

* Open triangle.c and make the following changes:

Below the line

```C
   #include "cube_texture_and_coords.h"
```

add

```C
   #include "omx_image.h"
```

Inside **init_textures** change where it loads the first texture from:

```C
   glBindTexture(GL_TEXTURE_2D, state->tex[0]);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, IMAGE_SIZE, IMAGE_SIZE, 0,
                GL_RGB, GL_UNSIGNED_BYTE, state->tex_buf1);
   glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, (GLfloat)GL_NEAREST);
   glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, (GLfloat)GL_NEAREST);
```
   
to:

```C
   doctor_chompers_t buffer;
   omx_image_fill_buffer_from_disk("Raspi_Colour_R.jpg", &buffer);
   omx_image_loader(state->tex[0], &state->display, &state->context, 0, 0, 0, 0, JPEG, &buffer);
   free(buffer.data);
```
   
* Run make and then ./hello_triangle.bin to see it run.

* The texture will probably be flipped. You can fix this by going into **cube_texture_and_coords.h**. Find the definition for **texCoords** and change the first entry from:

```
   0.f,  0.f,
   1.f,  0.f,
   0.f,  1.f,
   1.f,  1.f,
```

to

```
   0.f,  0.f,
   0.f,  1.f,
   1.f,  0.f,
   1.f,  1.f,
```
   
Run make clean and then make and run ./hello_triangle.bin

Try your own JPGs but just remember OpenMAX doesn't support them if they were saved as progressive JPGs. You can use PNGs here as well (change the omx_image_loader call parameter from JPEG to PNG), but PNGs have glitches if the height isn't divisible by 16. PNG does support the alpha channel though.

