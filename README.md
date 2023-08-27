# Random textures replacer API

Flexible textures replacer API written on ZScript (*ZDoom family engines). Internally uses a hash table and simple caching to quickly find a texture, so time complexity is a `O(1)` in the average case and `O(n)` in the worst case.

For the example see file "[Example.zsc](ZScript\RandomTexturesReplacer\Example.zsc)".


<br>


## Class `RandomTexturesReplacerBaseHandler`

Static event handler "`RandomTexturesReplacerBaseHandler`" is an abstract class for all other replacers. Inherits classes `RandomTexturesReplacerHandler_BaseRandom` and `RandomTexturesReplacerHandler_BaseLevelWide`.

There are no API fields for this class: all fields are internal, despite of the `protected` scope.

### API methods

```CPP
virtual void SetupCategories( void )
```
Method for categories initialization. In general its structure will be similar to the next code template:
```CPP
/* First category: */
AddCategory();

SetReplaceableTextures( ... );
ReplaceTo( ... );
ReplaceTo( ... );

SetReplaceableTextures( ... );
ReplaceTo( ... );

// <...>

/* Second category: */
AddCategory();

SetReplaceableTextures( ... );
ReplaceTo( ... );
ReplaceTo( ... );

// <...>
```
Does nothing by default.

<br>

```CPP
virtual void HandleWorldInit( WorldEvent e )
```
Just a `WorldLoaded` event after all internal initializations. Does nothing by default.

<br>

```CPP
void SetReplaceableTextures( String texlist, double chance = 1.0 )
```
Add a new source for the replace table.
 * `texlist`: comma-separated list of the replacee textures (e. g., "TEKGREN2,TEKGREN5,MARBLE2");
 * `chance`: chance to replace specified textures (from 0.0 to 1.0; out-of-bound values will be clamped to this range).

Method may be used several times in a row to specify different chances for different texture lists, but with the same replacer textures list.

<br>

```CPP
void ReplaceTo( String texlist, double relativechance = 1.0 )
```

Add a new destination for the replace table. May also be used several times in a row with different chances to create a less or greater group of replacer textures.
 * `texlist`: comma-separated list of the replacer textures (e. g., "TEKGREN1,TEKGREN2,TEKGREN4,SFALL1");
 * `relativechance`: chance relative to other texture lists in this groups. Applied to all textures in the first argument. Must be a positive value.

<br>

```CPP
protected uint AddCategory( int texmanType = -1, int texmanFlags = -1, class<Object> replaceTableClass = NULL )
```
Add a new replacing table category. Usually will be defined two or three categories: "walls"/"flats" or "walls"/"floors"/"ceilings". Also see "`SetupCategories()`" in "[Example.zsc](ZScript\RandomTexturesReplacer\Example.zsc)".
 * `texmanType`: any type from the enumeration `TexMan.EUseTypes`. By default a `TexMan.Type_Any` will be selected.
 * `texmanFlags`: same but for the texture manager flags. By default a `TexMan.TryAny` mask will be set.
 * `replaceTableClass`: non-standard `RandomTexturesReplacerReplaceTable` class. Will be set to default if argument is invalid.

Return a new category index in the internal dynamic array, starting from 0. Without external force will just increment it (0, 1, 2...). Automatically call a `SelectCategory()`.

<br>

```CPP
void ResetCategories( void )
```
Clear all of the saved categories and calls a virtual `SetupCategories()` again.

<br>

```CPP
void SelectCategory( uint nextcategory )
```
Set current category index to the `nextcategory` and performs some other internal actions.

<br>


## `RandomTexturesReplacerReplaceTable`

Class "`RandomTexturesReplacerReplaceTable`" is for selecting textures.

### API fields

```CPP
Array<TextureID> textures;		// All replacee textures for this replace group.
Array<double> chances;			// All chances for the replacee textures. chances.Size() == textures.Size().
double totalChance;				// Pre-calculated total chance sum.
TextureID const_NullTexture;	// Texture to return if no replaces must be done.
```

### API methods

```CPP
virtual void Init( void )
```
What to do right after internal initialization. Does nothing by default.

<br>

```CPP
protected TextureID tableRandomTexture( void )
```
Get a random texture from the list of the saved replacee textures. Time complexity is a `O(n)` in the average and worst cases.

<br>

```CPP
virtual TextureID GetReplacingTexture( void )
```
TextureID to return. By default calls `tableRandomTexture()`.
