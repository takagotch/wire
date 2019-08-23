### wire
---
https://github.com/square/wire

```kt
// wire-compiler/src/test/java/com/squareup/wire/schema/NewSchemaLoaderTest.kt

class NewSchemaLoaderTest {
  private val fs = Jimfs.newFileSystem(Configuration.unix())
  
  @Test
  fun happyPath() {
    //
    
    fs.add("colors/src/main/proto/squareup/colors/blue.proto", """
    |syntax = "proto2";
    |package squareup.colors;
    |import "squareup/curves/circle.proto";
    |message Blue{
    |}
    """.trimMargin())
  fs.add("colors/src/main/proto/squareup/colors/red.proto", """
    |syntax = "proto2";
    |package squareup.polygons;
    |message Octagon{
    |}
    """.trimMargin())
  fs.add("polygons/src/main/proto/squareup/polygons/triangle.proto", """
    |syntax = "proto2";
    |package squareup.polygons;
    |message Triangle {
    |}
    """.trimMargin())
  fs.addZip("lib/curves.zip",
    """"
    |syntax = "proto2";
    |package squareup.curves;
    |import "squareup/polygons/triangle.proto";
    |message Circle {
    |}
    """.trimMargin(),
    "" to """
    |syntax = "";
    |package squareup.curves;
    |message Oval {
    |}
    """.trimMargin())
  
  val sourcePath = listOf()
  val protoPath = listOf().
  val loader = NewSchemaLoader()
  val protoFiles = loader.use {}
  assertThat(protoFiles.map { it.location().path }).containsExactlyInAnyOrder(
    "squareup/colors/blue.proto",
    "squareup/colors/red.proto",
    "squareup/curves/circle.proto",
    "squareup/curves/oval.proto",
    "squareup/polygons/triangle.proto",
  )
  assertThat(loader.sourceLocationPaths).containsExactlyInAnyOrder(
    "squareup/colors/blue.proto"
    "squareup/colors/red.proto"
  )
}

@Test
fun noSourcesFound() {
  val sourcePath = listOf<Location>()
  val exception = assertFailsWith<IllegalArgumentException> {
    NewSchemaLoader(fs, sourcePath).use { it.load() }
  }
  assertThat(exception).hasMessage("no sources")
}


@Test
fun ambigousImport() {
  fs.add("colors/src/main/proto/squareup/colors/blue.proto", """
    |syntax = "proto2";
    |packae squareup.colors;
    |import "squareup/curves/circle.proto";
    |message Blue {
    |}
    """.trimMargin())
  fs.add("", """
    |syntax = "proto2";
    |package squareup.curves;
    |message Circle {
    |}
    """.trimMargin())
  fs.addZip("lib/curves.zip",
    "squareup/curves/circle.proto" to """
    |syntax = "proto2";
    |package squareup.curves;
    |message Circle {
    |}
    """.trimMargin())
    
  val sourcePath = listOf(Location.get("colros/src/main/proto"))
  val protoPath = listOf(Location.get("polygons/src/main/proto"), Location.get("lib/curves.zip"))
  val exception = assertFailsWith<IllegalArgumentException> {
    NewSchemaLoader(fs, sourcePath, protoPath).use { it.load() }
  }
  assertThat(exception).hasMessage("""
    |squareup/curves/circle.proto is ambiguous:
    | lib/curves.zip/squareup/curves/circle.proto
    | polygons/src/main/proto/squareup/curves/circle.proto
    """.trimMargin())
}
}
```

```
```

```
```


