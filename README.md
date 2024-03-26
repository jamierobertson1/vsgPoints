# vsgPoints - VulkanSceneGraph Point Cloud rendering

Cross platform, open source (MIT license) C++17 library and example set for rendering large point cloud data using VulkanSceneGraph. vsgPoints provides support for generating hierarchical LOD and paged LOD scene graph hierarchies to provide excellent performance and scalability. The support for database paging enables handling of very large point databases - over a billion point datasets run at a solid 60fps even on integrated GPUs.

To enable efficient use of main and GPU memory, point data is segmented into bricks and the source x, y, z values are quantized to 8bit, 10bit or 16bit values. Using the vsgPoints::Settings structure users can select the precision (default of 1mm) and bit representation they require, the higher precision and lower the brick count the smaller the bricks that will be used. The quantized data is packed into formats supported by graphics hardware enabling the vertex shaders to compute the final positions without any intermediate conversions.

To enable handling of point data with very large world coordinates, the data is translated to a local origin with a vsg::MatrixTransform used to place the rendered points in their proper world coordinate system.

The vsgPoints project contains a vsgPoints library that applications may link to if they wish to add point cloud loading and scene graph creation capabilities to their applications, and a vsgpoints_example utility program.

## System requirements/constraints

vsgPoints is usable, but still early in its development. Currently the reading and building of scene graphs is single threaded, and no attempt is made to minimize memory footprint, beyond quantization to 8/10/16bit. During read and scene graph generation of very large datasets you will need sufficient main memory, for billion points datasets you'll need 32GB+ main memory in order to build the scene graph.

For flat or LOD scene graphs, the GPU memory requirement loosely tracks the size of the scene graph in main memory/on disk so the larger it is, the more likely it will hit limits of available GPU memory. For very large point datasets it will be necessary to generate PagedLOD scene graphs which will result in creation of a hierarchy of PagedLOD nodes and associated files on disk, which can number in millions of files.  Paged databases perform very well, reducing the memory requirement substantially and enabling much larger datasets to be viewed at high framerates with the VulkanSceneGraph's multi-threaded database pager seamlessly handling the file loading as well as load balancing.

The downside with paged databases can be they stress the filesystem/OS more when you want to delete or move the directory hierarchy that has been generated by vsgPoints. In particular Windows can struggle when handling millions of files even on fast solid state disks, in particular if virus scanners are invoked during file operations.

## Roadmap

Possible features for future development:

* Multi-thread the reading of data and scene graph building,
* Saving intermediate results to avoid overloading main memory when building large point datasets and reduce memory footprint.
* Features like picking would also be nice features to add.

vsgPoints is open source, if you wish to see features developed then you are welcome to contribute time or funding to help make this happen.

## Dependencies

3rd party dependencies:

    C++17
    CMake
    VulkanSDK
    GLslang
    VulkanSceneGraph
    vsgXchange

## Building vsgPoints

Unix in source build:

    cd vsgPoints
    cmake .
    make -j 8
    make install

# vsgpoints_example application usage

To view .3dc or .asc point clouds, or .BIN (double x,y,z; uint8_t r, g, b) data :

~~~ sh
   vsgpoints_example mydata.asc
~~~~

To convert point cloud data to VulkanSceneGraph native format:

~~~ sh
   vsgpoints_example mydata.BIN -o mydata.vsgb
~~~~

Once you have converted to native .vsgb format you can load the data in any VulkanSceneGraph application:

~~~ sh
    vsgviewer mydata.vsgb
~~~

vsgpoints_example supports generating scene graphs in three ways, the command line options for these are:

| command line option | technique |
| --lod | using LOD's (the default) |
| --plod | Generation of paged databases |
| --flat |  flat group of point bricks |

~~~ sh
    # create vsg::LOD scene graph
    vsgpoints_example mydata.BIN --lod
    # or just rely upon the default being vsg::LOD scene graph
    vsgpoints_example mydata.BIN

    # create a flat group of point bricks
    vsgpoints_example mydata.BIN --flat

    # create a paged database, requires specification of output file
    vsgpoints_example mydata.BIN -o paged.vsgb --plod
~~~

vsgpoints_example also supports generating points from meshed models, which can be enabled with --mesh

~~~ sh
    # load a mesh model and convert to points as input rather than required loading of points data.
    vsgpoints_example mymodel.gltf --mesh
~~~

If you don't include an output filename using ~ -o filename.vsgb ~ then vsgpoints_example will automatically create a viewer to view the created scene graph, but if you output to a file no viewer will be created. If you still want the viewer to appear then add the -v option to force the viewer to be created.

To alter the precision and point size you can use the -p size_in_metres and --ps multiplier to control the precision (defaults to 0.001) and point size (defaults to 4 x precision).

~~~ sh
    # choose 5mm precision and ~50mm rendered point size (10 x 0.005)
    vsgpoints_example mydata.3dc -p 0.005 --ps 10
~~~
