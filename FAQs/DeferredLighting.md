Deferred Lighting
==========
The Deferred Lighting system lets you use real lights in your scene. Included light types are point, directional, spot and area lights.


Material Setup
==========
This is where using the deferred lighting system differs from your normal workflow when making a 2D game. In most games without realtime lighting you wont ever need to touch the Material system at all. Deferred lighting needs additional information (mainly normal maps) in order to calculate lighting so we have to dive into Nez's Material system. We won't get into actual asset create here so it is assumed that you know how to create (or auto-generate) your normal maps.

One neat trick the Nez deferrred lighting system has baked in is self illumination. Self illuminated portions of your texture dont need to have a light hit them for them to be visible. They are always lit. You can specify which portions of your texture should be self illuminated via the alpha channel of your normal map. A 0 value indicates no self illumination and a value of 1 indicates fully self illuminated. Don't forget to turn off premultiplied alpha in the Pipeline Tool when using self illumination! Nez needs that alpha channel intact to get the self illumination data. When using self illumination you have to let Nez know by setting `DeferredSpriteMaterial.setUseNormalAlphaChannelForSelfIllumination`. Additional control of self illumination at runtime is available by setting `DeferredSpriteMaterial.setSelfIlluminationPower`. Modulating the self illumination power lets you add some great atmosphere to a scene.

There are going to be times that you dont want your objects normal mapped or maybe you havent created your normal maps yet. The deferred lighting system can accommodate this as well. There is a built-in "null normal map texture" that you can configure in your Material that will make any object using it only take part in diffuse lighting. By default, `DeferredLightingRenderer.material` will be a Material containing the null normal map. Whenever a Renderer encounters a RenderableComponent with a null Material it will use it's own Material. What that all means is that if you add a RenderableComponent with a null Material it will be rendered with just diffuse lighting.

Below are the three most common Material setups: normal mapped lit, normal mapped lit self illuminated and only diffuse (no normal map).

```cs
// lit, normal mapped Material. normalMapTexture is a reference to the Texture2D that contains your normal map.
var standardMaterial = new DeferredSpriteMaterial( normalMapTexture );


// diffuse lit Material. The nullNormalMapTexture is used
var diffuseOnlylMaterial = new DeferredSpriteMaterial( deferredRenderer.nullNormalMapTexture );


// first we create the Material with our normal map. Note that our normal map should have an alpha channel for the self illumination and it
// needs to have premultiplied alpha disabled in the Pipeline Tool
var selfLitMaterial = new DeferredSpriteMaterial( selfLitNormalMapTexture );

// we can access the Effect on a Material<T> via the typedEffect property. We need to tell the Effect that we want self illumination and
// optionally set the self illumination power.
selfLitMaterial.typedEffect.setUseNormalAlphaChannelForSelfIllumination( true )
	.setSelfIlluminationPower( 0.5f );
```



Scene Setup
==========
There isn't much that needs to be done for our Scene setup. All we have to do is add a `DeferredLightingRenderer`. The values you pass to the constructor of this Renderer are very important though! You have to specify which renderLayer it should use for lights and which renderLayers contain your normal sprites.

```cs
// define your renderLayers somewhere easy to access
const int LIGHT_LAYER = 1;
const int OBJECT_LAYER1 = 10;
const int OBJECT_LAYER2 = 20

// add the DeferredLightingRenderer to your Scene specifying which renderLayer contains your lights and an arbitrary number of renderLayers for it to render
var deferredRenderer = scene.addRenderer( new DeferredLightingRenderer( 0, LIGHT_LAYER, OBJECT_LAYER1, OBJECT_LAYER2 ) );

// optionally set ambient lighting
deferredRenderer.setAmbientColor( Color.Black );
```

Now we just have to make sure that we use the proper renderLayers (easy to do since we were smart and made them const int) and Materials when creating our Renderables:

```cs
// create an Entity to house our sprite
var entity = createEntity( "sprite" );

// add a Sprite and here is the important part: be sure to set the renderLayer and material
entity.addComponent( new Sprite( spriteTexture ) )
	.setRenderLayer( OBJECT_LAYER1 )
	.setMaterial( standardMaterial );


// create an Entity to house our PointLight
var lightEntity = createEntity( "point-light" );

// add a PointLight Component and be sure to set the renderLayer to the lights layer!
lightEntity.addComponent( new PointLight( Color.Yellow ) )
	.setRenderLayer( LIGHT_LAYER );
```
