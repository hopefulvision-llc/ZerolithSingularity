---
tags:
  - zerolithsingularity
  - mathematical-specification
  - geometric-coordinate-system
  - wave-mechanics
  - dodecahedron-geometry
  - metatron-cube
  - 170-nodes
  - golden-ratio
  - sacred-geometry
  - technical-reference
backlinks:
  - "[[ZerolithSingularity]]"
  - "[[Resonance Engine]]"
  - "[[Sacred Geometry]]"
  - "[[120-Cell Mathematics]]"
version: "1.0"
date: December 2025
status: Reference Implementation
language: Python 3.10+
---

# ZerolithSingularity Mathematical Specification
## Geometric Coordinate System & Wave Mechanics v1.0

**Purpose**: Define precise mathematical foundation for the ZerolithSingularity architecture  
**Created**: December 2025  
**Status**: Reference Implementation  
**Language**: Python 3.10+

---

## Table of Contents

1. [Overview](#overview)
2. [Coordinate System](#coordinate-system)
3. [Dodecahedron Geometry](#dodecahedron-geometry)
4. [Metatron's Cube](#metatrons-cube)
5. [Node Coordinates](#node-coordinates)
6. [Adjacency Matrices](#adjacency-matrices)
7. [Wave Mechanics](#wave-mechanics)
8. [Convergence Functions](#convergence-functions)
9. [Python Implementation](#python-implementation)
10. [Validation Tests](#validation-tests)

---

## Overview

### Total Structure
- **170 nodes** in 3D space
- **12 dodecahedron faces** (hermetic pairs)
- **1 refraction face** (internal synthesis)
- **13 spheres per face** (Metatron's Cube)
- **1 singularity core** at origin

### Mathematical Constants
```python
PHI = (1 + sqrt(5)) / 2  # ≈ 1.618034 (golden ratio)
PI = 3.14159265359
```

---

## Coordinate System

### Origin & Scaling
- **Origin**: (0, 0, 0) = Singularity Core
- **Unit**: Dodecahedron inscribed in unit sphere (R=1)
- **Precision**: Float64

### Node Identification
Format: `(face_index, sphere_index)`
- Face: 0-11 (dodecahedron), 12 (refraction)
- Sphere: 0-12 within each face
- Example: `(0, 0)` = Crown on Face 0
- Core: `(12, 12)` = (0, 0, 0)

---

## Dodecahedron Geometry

### Vertex Calculation

Regular dodecahedron vertices for unit sphere:

**Cube vertices** (8):
```python
vertices_cube = [
    (±1, ±1, ±1)  # All 8 combinations
]
```

**Golden rectangle vertices** (12):
```python
vertices_rect = [
    (0, ±1/PHI, ±PHI),
    (±1/PHI, ±PHI, 0),
    (±PHI, 0, ±1/PHI)
]
```

Total: 20 vertices

### Python Implementation

```python
import numpy as np

PHI = (1 + np.sqrt(5)) / 2

def dodecahedron_vertices():
    """Generate all 20 vertices of unit dodecahedron"""
    vertices = []
    
    # Cube vertices
    for i in [-1, 1]:
        for j in [-1, 1]:
            for k in [-1, 1]:
                vertices.append((i, j, k))
    
    # Golden rectangle vertices
    for perm in [(0, 1/PHI, PHI), (1/PHI, PHI, 0), (PHI, 0, 1/PHI)]:
        for signs in [(1,1,1), (1,1,-1), (1,-1,1), (1,-1,-1),
                      (-1,1,1), (-1,1,-1), (-1,-1,1), (-1,-1,-1)]:
            if all(p != 0 for p in perm):  # Skip duplicates
                vertex = tuple(p*s for p,s in zip(perm, signs))
                if vertex not in vertices:
                    vertices.append(vertex)
    
    return np.array(vertices)
```

### Face Centers

Each pentagonal face center = average of 5 vertices:

```python
def dodecahedron_face_centers():
    """Calculate centers of all 12 pentagonal faces"""
    # Face vertex indices (which 5 vertices form each face)
    faces = get_dodecahedron_faces()  # Returns list of vertex index groups
    vertices = dodecahedron_vertices()
    
    centers = []
    for face_vertices in faces:
        center = np.mean([vertices[i] for i in face_vertices], axis=0)
        centers.append(center)
    
    return np.array(centers)
```

Expected face center distance from origin: ≈ 0.7947

---

## Metatron's Cube

### 2D Layout (Standard)

13 spheres in hexagonal pattern:

**Outer ring** (6 spheres):
```python
def metatron_outer_ring():
    """Hexagon vertices at radius 1"""
    angles = np.linspace(0, 2*np.pi, 7)[:-1]  # 6 points
    return [(np.cos(a), np.sin(a)) for a in angles]
```

Spheres 0-5 at angles: 0°, 60°, 120°, 180°, 240°, 300°

**Inner ring** (6 spheres):
```python
def metatron_inner_ring():
    """Hexagon vertices at radius 0.5"""
    angles = np.linspace(30, 390, 7)[:-1] * (np.pi/180)  # Offset 30°
    return [(0.5*np.cos(a), 0.5*np.sin(a)) for a in angles]
```

Spheres 6-11 at angles: 30°, 90°, 150°, 210°, 270°, 330°

**Center** (1 sphere):
```python
Sphere 12: (0, 0)
```

### Scaling for Faces

```python
MC_SCALE = 0.3  # Scale factor (30% of dodecahedron radius)

def scale_metatron(point_2d):
    """Scale 2D Metatron point"""
    return (point_2d[0] * MC_SCALE, point_2d[1] * MC_SCALE)
```

### 3D Transformation

Transform 2D Metatron's Cube onto 3D dodecahedron face:

```python
def transform_to_face(mc_point_2d, face_center, face_normal):
    """
    Transform 2D Metatron point to 3D face position
    
    Args:
        mc_point_2d: (x, y) in 2D Metatron space
        face_center: (x, y, z) center of dodecahedron face
        face_normal: (nx, ny, nz) outward normal of face
    
    Returns:
        (x, y, z) in 3D space
    """
    # Scale the 2D point
    x, y = scale_metatron(mc_point_2d)
    
    # Create local coordinate system on the face
    # normal = z-axis of local system
    normal = face_normal / np.linalg.norm(face_normal)
    
    # Find perpendicular vectors for x and y axes
    # Use cross product with (0,0,1) unless normal is too close to it
    if abs(normal[2]) < 0.9:
        tangent1 = np.cross(normal, [0, 0, 1])
    else:
        tangent1 = np.cross(normal, [1, 0, 0])
    
    tangent1 = tangent1 / np.linalg.norm(tangent1)
    tangent2 = np.cross(normal, tangent1)
    tangent2 = tangent2 / np.linalg.norm(tangent2)
    
    # Transform: local (x,y,0) to global 3D
    point_3d = face_center + x * tangent1 + y * tangent2
    
    return tuple(point_3d)
```

---

## Node Coordinates

### Primary Nodes (Faces 0-11)

```python
def calculate_primary_nodes():
    """Calculate coordinates for all 156 primary nodes"""
    nodes = {}
    face_centers = dodecahedron_face_centers()
    face_normals = dodecahedron_face_normals()
    
    for face_idx in range(12):
        for sphere_idx in range(13):
            # Get 2D Metatron position
            mc_2d = get_metatron_sphere_2d(sphere_idx)
            
            # Transform to 3D
            pos_3d = transform_to_face(
                mc_2d,
                face_centers[face_idx],
                face_normals[face_idx]
            )
            
            nodes[(face_idx, sphere_idx)] = pos_3d
    
    return nodes
```

### Refraction Face (Face 12)

Internal layer at intermediate radius:

```python
def calculate_refraction_nodes():
    """Calculate coordinates for 13 refraction nodes"""
    nodes = {}
    REFRACTION_RADIUS = 0.4  # Between faces (~0.79) and center (0)
    
    # Use same Metatron pattern, scaled to intermediate sphere
    for sphere_idx in range(13):
        mc_2d = get_metatron_sphere_2d(sphere_idx)
        
        # Scale and convert to 3D on sphere of radius REFRACTION_RADIUS
        # Align with Face 0 orientation for consistency
        if sphere_idx == 12:  # Center sphere
            pos_3d = (0, 0, 0)  # This IS the Singularity Core
        else:
            # Distribute on intermediate sphere
            theta = 2 * np.pi * sphere_idx / 12
            x = REFRACTION_RADIUS * np.cos(theta) * mc_2d[0]
            y = REFRACTION_RADIUS * np.sin(theta) * mc_2d[1]
            z = REFRACTION_RADIUS * 0.3 * (sphere_idx % 3 - 1)  # Slight z variation
            pos_3d = (x, y, z)
        
        nodes[(12, sphere_idx)] = pos_3d
    
    return nodes
```

### Complete Coordinate Set

```python
def calculate_all_nodes():
    """Generate all 170 node coordinates"""
    nodes = {}
    nodes.update(calculate_primary_nodes())  # 156 nodes
    nodes.update(calculate_refraction_nodes())  # 14 nodes (13 + core)
    
    # Verify singularity core
    assert nodes[(12, 12)] == (0, 0, 0)
    
    return nodes
```

---

## Adjacency Matrices

### Lateral Adjacency (Within Face)

Metatron's Cube connection pattern:

```python
METATRON_ADJACENCY = {
    0: [1, 5, 6, 11, 12],
    1: [0, 2, 6, 7, 12],
    2: [1, 3, 7, 8, 12],
    3: [2, 4, 8, 9, 12],
    4: [3, 5, 9, 10, 12],
    5: [0, 4, 10, 11, 12],
    6: [0, 1, 7, 11, 12],
    7: [1, 2, 6, 8, 12],
    8: [2, 3, 7, 9, 12],
    9: [3, 4, 8, 10, 12],
    10: [4, 5, 9, 11, 12],
    11: [0, 5, 6, 10, 12],
    12: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
}
```

### Cross-Face Adjacency

Spheres on adjacent faces may be close enough for interaction:

```python
def cross_face_adjacency(nodes, threshold=0.35):
    """
    Find cross-face adjacent nodes
    
    Args:
        nodes: dict of {(face, sphere): (x,y,z)}
        threshold: max distance for adjacency
    
    Returns:
        dict: {node: [adjacent_nodes_on_other_faces]}
    """
    cross_adj = {node: [] for node in nodes}
    
    for node1 in nodes:
        for node2 in nodes:
            if node1[0] == node2[0]:  # Same face
                continue
            
            dist = euclidean_distance(nodes[node1], nodes[node2])
            if dist <= threshold:
                cross_adj[node1].append(node2)
    
    return cross_adj
```

### Radial Connections

Every node → Singularity Core:

```python
def radial_adjacency(nodes):
    """All nodes connect to (12,12)"""
    radial = {node: [(12, 12)] for node in nodes if node != (12, 12)}
    radial[(12, 12)] = list(nodes.keys())[:-1]  # Core connects to all
    return radial
```

### Complete Adjacency Matrix

```python
def build_adjacency_matrix(nodes):
    """
    Create 170×170 adjacency matrix
    
    Returns:
        np.array: Binary matrix where [i,j]=1 if connected
    """
    node_list = sorted(nodes.keys())
    n = len(node_list)
    matrix = np.zeros((n, n), dtype=int)
    
    # Map node to index
    node_to_idx = {node: i for i, node in enumerate(node_list)}
    
    # Lateral connections (same face)
    for node in nodes:
        face_idx, sphere_idx = node
        i = node_to_idx[node]
        
        for adj_sphere in METATRON_ADJACENCY[sphere_idx]:
            adj_node = (face_idx, adj_sphere)
            if adj_node in nodes:
                j = node_to_idx[adj_node]
                matrix[i, j] = 1
    
    # Cross-face connections
    cross_adj = cross_face_adjacency(nodes)
    for node, adj_list in cross_adj.items():
        i = node_to_idx[node]
        for adj_node in adj_list:
            j = node_to_idx[adj_node]
            matrix[i, j] = 1
    
    # Radial connections (all → core)
    core_idx = node_to_idx[(12, 12)]
    matrix[:, core_idx] = 1  # All connect to core
    matrix[core_idx, :] = 1  # Core connects to all
    matrix[core_idx, core_idx] = 0  # Core doesn't connect to itself
    
    return matrix
```

---

## Wave Mechanics

### Wave Function

Each sphere generates consciousness wave:

```python
def sphere_wave(sphere_type, position, time, phi=1.618034):
    """
    Calculate wave amplitude at time t
    
    Args:
        sphere_type: 0-12 (Crown, Wisdom, etc.)
        position: (x, y, z)
        time: current time step
        phi: golden ratio
    
    Returns:
        complex: wave amplitude
    """
    # Frequency based on sphere type
    base_freq = phi ** (sphere_type / 13.0)
    
    # Spatial decay
    r = np.linalg.norm(position)
    spatial = np.exp(-r / phi)
    
    # Temporal oscillation
    temporal = np.exp(2j * np.pi * base_freq * time)
    
    return spatial * temporal
```

### Interference

When waves meet:

```python
def interference_intensity(wave1, wave2):
    """
    Calculate interference between two waves
    
    Returns:
        float: normalized intensity (0-1)
    """
    # Superposition
    combined = wave1 + wave2
    intensity = abs(combined) ** 2
    
    # Normalize by maximum possible
    max_intensity = (abs(wave1) + abs(wave2)) ** 2
    return intensity / max_intensity if max_intensity > 0 else 0
```

### Propagation Simulation

```python
def simulate_wave_propagation(nodes, adjacency, source, steps=100):
    """
    Simulate wave spreading through network
    
    Args:
        nodes: node coordinates
        adjacency: adjacency matrix
        source: (face, sphere) starting node
        steps: number of time steps
    
    Returns:
        dict: {node: [amplitudes over time]}
    """
    node_list = sorted(nodes.keys())
    node_to_idx = {n: i for i, n in enumerate(node_list)}
    
    # Initialize
    state = np.zeros(len(nodes))
    state[node_to_idx[source]] = 1.0
    
    history = {node: [] for node in nodes}
    
    for t in range(steps):
        # Record current state
        for node in nodes:
            history[node].append(state[node_to_idx[node]])
        
        # Propagate
        new_state = np.zeros(len(nodes))
        for i, node in enumerate(node_list):
            # Current value
            current = state[i]
            
            # Sum from neighbors
            neighbors = np.where(adjacency[i, :] == 1)[0]
            incoming = sum(state[j] for j in neighbors)
            
            # Update with damping
            damping = 0.95
            new_state[i] = damping * (current + 0.1 * incoming)
        
        state = new_state
    
    return history
```

---

## Convergence Functions

### Radial Convergence

```python
def convergence_vector(node_pos, center=(0,0,0)):
    """
    Unit vector pointing toward center
    
    Returns:
        (dx, dy, dz): unit vector
    """
    vec = np.array(center) - np.array(node_pos)
    magnitude = np.linalg.norm(vec)
    return tuple(vec / magnitude) if magnitude > 0 else (0, 0, 0)
```

### Synthesis Transform

Refraction face distills from primary faces:

```python
def synthesis_value(primary_data, sphere_type):
    """
    Calculate synthesized value for refraction sphere
    
    Args:
        primary_data: {(face, sphere): value} for faces 0-11
        sphere_type: which sphere position (0-12)
    
    Returns:
        float: synthesized value
    """
    # Collect values from this sphere type across all 12 faces
    values = [
        primary_data.get((face, sphere_type), 0.0)
        for face in range(12)
    ]
    
    # Synthesis (weighted average with amplification)
    synthesis = sum(values) / len(values)
    amplification = 1.2  # Refraction intensifies
    
    return synthesis * amplification
```

### Singularity Integration

All converge to one:

```python
def singularity_emergence(all_values):
    """
    Calculate unified consciousness at core
    
    Args:
        all_values: {(face, sphere): value} for all 169 input nodes
    
    Returns:
        float: unified consciousness value
    """
    # Exclude core itself
    inputs = [v for (f,s), v in all_values.items() if (f,s) != (12,12)]
    
    # Calculate coherence
    mean = np.mean(inputs)
    std = np.std(inputs)
    coherence = max(0, 1 - std)  # High coherence = low variance
    
    # Emergence with non-linear amplification
    emergence_factor = 1.0 + (coherence ** 2)
    unified = mean * emergence_factor
    
    return unified
```

---

## Python Implementation

### Complete Module Structure

```
zerolithsingularity/
├── __init__.py
├── geometry.py       # Dodecahedron & Metatron's Cube
├── nodes.py          # Node coordinate calculation
├── adjacency.py      # Connection matrices
├── waves.py          # Wave mechanics
├── convergence.py    # Synthesis & singularity
├── visualization.py  # Export for Three.js
└── tests/
    └── test_all.py   # Validation tests
```

### Main Class

```python
# zerolithsingularity/__init__.py

from .geometry import *
from .nodes import *
from .adjacency import *
from .waves import *
from .convergence import *

class ZerolithSingularity:
    def __init__(self):
        self.nodes = calculate_all_nodes()
        self.adjacency = build_adjacency_matrix(self.nodes)
    
    def get_node(self, face, sphere):
        """Get coordinates of specific node"""
        return self.nodes.get((face, sphere))
    
    def propagate_wave(self, source, steps=100):
        """Simulate wave from source"""
        return simulate_wave_propagation(
            self.nodes,
            self.adjacency,
            source,
            steps
        )
    
    def export_json(self, filename):
        """Export for Three.js visualization"""
        data = {
            'nodes': [
                {
                    'id': f"{f}_{s}",
                    'face': f,
                    'sphere': s,
                    'position': list(pos)
                }
                for (f, s), pos in self.nodes.items()
            ],
            'adjacency': self.adjacency.tolist()
        }
        
        import json
        with open(filename, 'w') as file:
            json.dump(data, file, indent=2)
```

### Usage

```python
# example.py

from zerolithsingularity import ZerolithSingularity

# Create instance
zs = ZerolithSingularity()

# Get specific node
crown = zs.get_node(0, 0)
print(f"Crown position: {crown}")

# Simulate wave
history = zs.propagate_wave(source=(0, 0), steps=100)

# Export for visualization
zs.export_json("zs_data.json")
```

---

## Validation Tests

### Test 1: Node Count

```python
def test_node_count():
    zs = ZerolithSingularity()
    assert len(zs.nodes) == 170
```

### Test 2: Singularity at Origin

```python
def test_singularity_position():
    zs = ZerolithSingularity()
    core = zs.get_node(12, 12)
    assert core == (0, 0, 0)
```

### Test 3: Adjacency Symmetry

```python
def test_adjacency_symmetric():
    zs = ZerolithSingularity()
    assert np.array_equal(zs.adjacency, zs.adjacency.T)
```

### Test 4: All Connect to Core

```python
def test_radial_connections():
    zs = ZerolithSingularity()
    node_list = sorted(zs.nodes.keys())
    core_idx = node_list.index((12, 12))
    
    # All nodes should connect to core
    for i in range(len(node_list)):
        if i != core_idx:
            assert zs.adjacency[i, core_idx] == 1
```

### Test 5: Wave Conservation

```python
def test_wave_propagation():
    zs = ZerolithSingularity()
    history = zs.propagate_wave((0, 0), steps=50)
    
    # Energy should be conserved (approximately)
    initial_energy = sum(h[0]**2 for h in history.values())
    final_energy = sum(h[-1]**2 for h in history.values())
    
    # With damping, final < initial
    assert final_energy < initial_energy
```

---

## Next Steps

### Implementation Roadmap

**Week 1-2**: Core Implementation
- [ ] Implement `geometry.py`
- [ ] Implement `nodes.py`
- [ ] Implement `adjacency.py`
- [ ] Run validation tests

**Week 3-4**: Wave Mechanics & Visualization
- [ ] Implement `waves.py`
- [ ] Implement `convergence.py`
- [ ] Create Three.js visualization
- [ ] Generate demo animations

**Q1 2026**: Refinement & Documentation
- [ ] Community feedback integration
- [ ] Performance optimization
- [ ] Comprehensive documentation
- [ ] Tutorial creation

---

*ZerolithSingularity Mathematical Specification v1.0*  
*December 2025*  
*HopefulVision LLC*

**"Sacred Geometry Meets Executable Mathematics"**
