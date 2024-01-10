
## Multiresolution mesh editing
For this task, you will compute a mesh deformation based on the rotations and translations applied interactively to a subset of its vertices via the mouse. Let ![](https://latex.codecogs.com/svg.latex?{H}) be the set of "handle" vertices that the user can manipulate (or leave fixed). We want to compute a deformation for the remaining vertices, denoted as ![](https://latex.codecogs.com/svg.latex?R).

Let ![](https://latex.codecogs.com/svg.latex?{\mathcal{S}}) be our input surface, represented as a triangle mesh. We want to compute a new surface that contains:
- the vertices in ![](https://latex.codecogs.com/svg.latex?&space;H) translated/rotated using the user-provided transformation ![](https://latex.codecogs.com/svg.latex?&space;t), and
- the vertices in ![](https://latex.codecogs.com/svg.latex?&space;R) properly deformed using the algorithm described next.

The algorithm is divided in three phases:

1. removing high-frequency details,
2. deforming the smooth mesh, and
3. transferring high-frequency details to the deformed surface.

![](img/schema.jpg)
*Fig. 1: Algorithm Overview*

### Selecting the handles
*Implementation for handle painting provided by prof. Teseo Schneider*
A minimal sphere-based interface for selecting vertices has been implemented in `Selection.ipynb`.
To use it, execute the cell with widgets, and select numbered keys to specify the segment index (`+` key to add more).
Use sliders to adjust the position and radius of the selector sphere, and `Paint!` to mark the current segment.
Finally, the accepted segments are stored in the variable `segments_recording`,
which you can save as `npy` file for the next tasks. There are several prepared file with suffix `.label.npy` in the `data/` folder.

*Note*: the array `seg` stored is of integer type, with values 0,1,2,... where `seg==0` indicates the free vertices, `seg==1` means the first selected segment and so on.

In `Assignment5.ipynb`, another interface is provided to specify the displacement and rotation.
The dropdown menu specifies which segement is being manipulated, and the slider
specifies the displacement `x,y,z` and rotation `α,β,γ`, the updated vertex positions are stored in `handle_vertex_positions`.

*Note*: within `handle_vertex_positions`, only the entries corresponding to selections (`seg>0`) are supposed to be used.


### Step 1: Removal of high-frequency details

<img align="left" width="200" src="img/hand.png">
<img align="left" width="200" src="img/hand_b.png">
<br clear="both"/>
<img align="left" width="200" src="img/woody.png">
<img align="left" width="200" src="img/woody_b.png">
<br clear="both"/>

*Fig. 2: Input and Smoothed Meshes*

We remove the high-frequency details from the vertices ![](https://latex.codecogs.com/svg.latex?{R}) in ![](https://latex.codecogs.com/svg.latex?{\mathcal{S}}) by minimizing the thin-plate energy, which involves solving a bi-Laplacian system arising from the quadratic energy minimization:

<img align="center" width="200" src="https://i.imgur.com/6IRzdBj.png">
<!--
\begin{aligned} \min_\textbf{v} & \quad\textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = \textbf{o}_H,
\end{aligned}
-->

where ![](https://latex.codecogs.com/svg.latex?\textbf{o}_H) are the handle ![](https://latex.codecogs.com/svg.latex?H)'s vertex positions, ![](https://latex.codecogs.com/svg.latex?\textbf{L}_\omega) is the cotan Laplacian of ![](https://latex.codecogs.com/svg.latex?\mathcal{S}), and ![](https://latex.codecogs.com/svg.latex?\textbf{M}) is the mass matrix of ![](https://latex.codecogs.com/svg.latex?\mathcal{S}).
Notice that ![](https://latex.codecogs.com/svg.latex?\textbf{L}_\omega) is the symmetric matrix consisting of the cotangent weights ONLY (without the division by Voronoi areas). In other words, it evaluates an "integrated" Laplacian rather than an "averaged" laplacian when applied to a vector of vertices. The inverse mass matrix appearing in the formula above then applies the appropriate rescaling so that the laplacian operator can be applied again (i.e., so that the Laplacian value computed at each vertex can be interpreted as a piecewise linear scalar field whose Laplacian can be computed).
This optimization will produce a mesh similar to the one in Figure 2. Note that the part of the surface that we want to deform is now free of high-frequency details. We call this mesh ![](https://latex.codecogs.com/svg.latex?\mathcal{B}).


*Relevant `scipy` functions:* `scipy.sparse.csc_matrix`, `scipy.sparse.diags`,


### Step 2: Deforming the smooth mesh
<img align="left" width="200" src="img/hand_t.png">
<img align="left" width="200" src="img/woody_t.png">
<br clear="both"/>

*Fig. 3: Deformed/Smoothed Meshes*

The new deformed mesh is computed similarly to the previous step, by solving the minimization:<br/>
<img align="center" width="200" src="https://i.imgur.com/xv8ZcsA.png">
<!-- $$
\begin{aligned} \min_\textbf{v}& \quad \textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = t(\textbf{o}_H),
\end{aligned}
$$ -->
where ![](https://latex.codecogs.com/svg.latex?t(\textbf{o}_H)) are the new handle vertex positions after applying the user's transformation. We call this mesh ![](https://latex.codecogs.com/svg.latex?\mathcal{B}').

*Relevant `scipy` functions:* `scipy.sparse.linalg.spsolve`


### Step 3: Transferring high-frequency details to the deformed surface
<img align="left" width="200" src="img/hand_bd.png">
<img align="left" width="200" src="img/hand_td.png">
<br clear="both"/>
<img align="left" width="200" src="img/woody_bd.png">
<img align="left" width="200" src="img/woody_td.png">
<br clear="both"/>

*Fig 4: Displacements on ![](https://latex.codecogs.com/svg.latex?\mathcal{B}) (left) and ![](https://latex.codecogs.com/svg.latex?\mathcal{B}') (right)*

The high-frequency details on the original surface are extracted from ![](https://latex.codecogs.com/svg.latex?\mathcal{S}) and transferred to ![](https://latex.codecogs.com/svg.latex?\mathcal{B}'). We first encode the high-frequency details of ![](https://latex.codecogs.com/svg.latex?\mathcal{S}) as displacements w.r.t. ![](https://latex.codecogs.com/svg.latex?\mathcal{B}).
We define an orthogonal reference frame on every vertex ![](https://latex.codecogs.com/svg.latex?v) of ![](https://latex.codecogs.com/svg.latex?\mathcal{B}) using:
1. The unit vertex normal
2. The normalized projection of one of ![](https://latex.codecogs.com/svg.latex?v)'s outgoing edges onto the tangent plane defined by the vertex normal. A stable choice is the edge whose projection onto the tangent plane is longest.
3. The cross-product between (1) and (2)

For every vertex ![](https://latex.codecogs.com/svg.latex?v), we compute the displacement vector that takes ![](https://latex.codecogs.com/svg.latex?v) from ![](https://latex.codecogs.com/svg.latex?\mathcal{B}) to ![](https://latex.codecogs.com/svg.latex?\mathcal{S}) and represent it as a vector in ![](https://latex.codecogs.com/svg.latex?v)'s reference frame.
For every vertex of ![](https://latex.codecogs.com/svg.latex?\mathcal{B}'), we also construct a reference frame using the normal and the SAME outgoing edge we selected for ![](https://latex.codecogs.com/svg.latex?\mathcal{B}) (not the longest in ![](https://latex.codecogs.com/svg.latex?\mathcal{B}'); it is important that the edges used to build both reference frames are the same). We can now use the displacement vector components computed in the previous paragraph to define transferred displacement vectors in the new reference frames of ![](https://latex.codecogs.com/svg.latex?\mathcal{B}'). See Figure 4 for an example.
Applying the transferred displacements to the vertices of ![](https://latex.codecogs.com/svg.latex?\mathcal{B}') generates the final deformed mesh ![](https://latex.codecogs.com/svg.latex?\mathcal{S}'). See Figure 5 for an example.

<img align="left" width="200" src="img/hand_f.png">
<img align="left" width="200" src="img/woody_f.png">
<br clear="both"/>

*Fig 5: Final Deformation Results*

Recommended outputs:
- Provide plots for 4 different deformed meshes. For each example, provide a rendering of ![](https://latex.codecogs.com/svg.latex?\mathcal{S}), ![](https://latex.codecogs.com/svg.latex?\mathcal{B}), ![](https://latex.codecogs.com/svg.latex?\mathcal{B}') and ![](https://latex.codecogs.com/svg.latex?\mathcal{S}').


