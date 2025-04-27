# 🧵 Minecraft Mini World Gen – Multithreading Exercise

Welcome to the project for the **ADCREA "Threading and Parallel Execution"** lecture at HSLU! 🎮

---

## 📚 What This Project Contains

This Unity 6 project is a **simple Minecraft-style world generator**, featuring:

- Procedural **terrain generation** using Perlin noise
- Dynamic **chunk loading and unloading** based on player position
- Basic **block placement** and **block removal**
- **Chunk meshing** to turn block data into 3D meshes
- Basic **structure generation** (trees)

---

## 🛑 Current Problem

Currently, **everything happens on the Main Thread**:

- Terrain generation
- Structure generation
- Mesh building
- Updating GameObjects

As a result:
- ❌ The game **freezes** when generating or loading chunks.
- ❌ **FPS drops** dramatically when moving into new areas.
- ❌ **Unplayable** experience in larger worlds.

---

## 🎯 Your Mission

Your task is to **multithread** the world generation and mesh creation to make the gameplay **smooth and responsive**.

✅ Move heavy calculations to **background threads**.  
✅ Keep Unity GameObject access **only on the Main Thread**.  
✅ Avoid **blocking the Main Thread** whenever possible.  
✅ Maintain **correct game logic** — chunks should still load properly.

# 💡 Hints (expand one by one!)

> ❗ Try solving it yourself first — only open a hint if you're stuck!

---

### 🧩 Hint 1: Where does the heavy freeze happen?
- Look inside the file `ChunkMeshCreator.cs`.
- Focus especially on the method called `CreateMeshFromData(...)`.
- **This is where most of the heavy work is happening.**

---

### 🧩 Hint 2: What inside `CreateMeshFromData` causes freezing?
- Look at the three nested loops:  
  `for (int x = ...) { for (int y = ...) { for (int z = ...) { ... } } }`
- These loops generate **all vertices and faces** for one chunk.
- Doing this **on the main thread** blocks Unity from rendering or accepting player input.

---

### 🧩 Hint 3: How could you make this heavy work run smoother?
- We need to **move** all the heavy work **into a background thread**.
- This will let Unity’s main thread stay responsive.
- No lag, no stutter — smoother gameplay!

---

### 🧩 Hint 4: Which C# tool makes this easy?
- Use a **`Task`** to run code in the background.
- `Task.Run(() => { /* heavy work */ });`  
- Combine it with `await` inside an `async` method to know when it's done.

---

### 🧩 Hint 5: Very Specific Guidance
- Change the method signature:  
  ➔ From `IEnumerator CreateMeshFromData(...)`  
  ➔ To `async IEnumerator CreateMeshFromData(...)`
- Inside the method:
  - Wrap the **entire block of slow work** (all `for` loops **AND** the `Thread.Sleep(...)`) inside a `Task.Run`.
  - Nothing heavy should stay outside!

---
  
### 🧩 Final Hint (Solution! 🚀)

Here’s the structure you want:

```csharp
public IEnumerator CreateMeshFromData(int[,,] Data, System.Action<Mesh> callback)
{
    List<Vector3> Vertices = new List<Vector3>();
    List<int> Indices = new List<int>();
    List<Vector2> UVs = new List<Vector2>();
    Mesh m = new Mesh();

    Task t = Task.Factory.StartNew(delegate
    {
        for (int x = 0; x < WorldGenerator.ChunkSize.x; x++)
        {
            for (int y = 0; y < WorldGenerator.ChunkSize.y; y++)
            {
                for (int z = 0; z < WorldGenerator.ChunkSize.z; z++)
                {
                    Vector3Int BlockPos = new Vector3Int(x, y, z);
                    for (int i = 0; i < CheckDirections.Length; i++)
                    {
                        Vector3Int BlockToCheck = BlockPos + CheckDirections[i];

                        try
                        {
                            if (Data[BlockToCheck.x, BlockToCheck.y, BlockToCheck.z] == 0)
                            {
                                if (Data[BlockPos.x, BlockPos.y, BlockPos.z] != 0)
                                {
                                    int CurrentBlockID = Data[BlockPos.x, BlockPos.y, BlockPos.z];
                                    TextureLoader.CubeTexture TextureToApply = TextureLoaderInstance.Textures[CurrentBlockID];
                                    FaceData FaceToApply = CubeFaces[CheckDirections[i]];

                                    foreach (Vector3 vert in FaceToApply.Vertices)
                                    {
                                        Vertices.Add(new Vector3(x, y, z) + vert);
                                    }

                                    foreach (int tri in FaceToApply.Indices)
                                    {
                                        Indices.Add(Vertices.Count - 4 + tri);
                                    }

                                    Vector2[] UVsToAdd = TextureToApply.GetUVsAtDirectionT(CheckDirections[i]);
                                    foreach (int UVIndex in FaceToApply.UVIndexOrder)
                                    {
                                        UVs.Add(UVsToAdd[UVIndex]);
                                    }
                                }
                            }
                        }
                        catch (System.Exception)
                        {
                            //Draws faces towards the outside of the data
                            if (Data[BlockPos.x, BlockPos.y, BlockPos.z] != 0)
                            {
                                int CurrentBlockID = Data[BlockPos.x, BlockPos.y, BlockPos.z];
                                TextureLoader.CubeTexture TextureToApply = TextureLoaderInstance.Textures[CurrentBlockID];
                                FaceData FaceToApply = CubeFaces[CheckDirections[i]];

                                foreach (Vector3 vert in FaceToApply.Vertices)
                                {
                                    Vertices.Add(new Vector3(x, y, z) + vert);
                                }

                                foreach (int tri in FaceToApply.Indices)
                                {
                                    Indices.Add(Vertices.Count - 4 + tri);
                                }

                                Vector2[] UVsToAdd = TextureToApply.GetUVsAtDirectionT(CheckDirections[i]);
                                foreach (int UVIndex in FaceToApply.UVIndexOrder)
                                {
                                    UVs.Add(UVsToAdd[UVIndex]);
                                }
                            }
                        }
                    }
                }
            }
        }
    });

    yield return new WaitUntil(() => {
        return t.IsCompleted || t.IsCanceled;
    });

    if (t.Exception != null)
        Debug.LogError(t.Exception);

    m.SetVertices(Vertices);
    m.SetIndices(Indices, MeshTopology.Triangles, 0);
    m.SetUVs(0, UVs);

    m.RecalculateBounds();
    m.RecalculateTangents();
    m.RecalculateNormals();

    callback(m);
}
```

✅ Now heavy work happens in the background  
✅ Unity stays smooth and playable  
✅ Chunks generate **without freezing** the game!
