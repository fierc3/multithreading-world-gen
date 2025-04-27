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