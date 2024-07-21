Final project for Geometric modeling course

## Multiresolution mesh editing
Computed a mesh deformation based on interactively applied rotations and translations to a subset of the mesh's vertices. Let *S* be out input surface, *H* be the subset of *S*  defining the "handle" vertices that the user can manipulate (or leave fixed) and *R* be the remaining vertices in *S* . 

We then want to compute a new surface that contains:

- the verices in *H* translated/rotated using the user-provided transformation *t*, and
- the vertices in *R* properly deformed.

The deformation algorithm is divided into three phases:

1. removal of high-frequency details
2. deforming the smooth mesh
3. transfer of high-frequency details to the deformed surface

![](img/schema.jpg)

### Removal of high-frequency details

<img align="left" width="200" src="img/hand.png">
<img align="left" width="200" src="img/hand_b.png">
<br clear="both"/>
<img align="left" width="200" src="img/woody.png">
<img align="left" width="200" src="img/woody_b.png">
<br clear="both"/>

Removal of the high-frequency details from the vertices *R* in *S* is accomplished by the minimization of thin-plate energy, which involves solving the bi-Laplacian system arising from the quadratic energy minimization:

<img align="center" width="200" src="https://i.imgur.com/6IRzdBj.png">
<!--
\begin{aligned} \min_\textbf{v} & \quad\textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = \textbf{o}_H,
\end{aligned}
-->

where **O**<sub>*H*</sub> are the handle *H*'s vertex positions, **L**<sub>*w*</sub> is the cotan Laplacian of ***S***, and **M** is the mass matrix of ***S***.  Notice that **L**<sub>*w*</sub> is the symmetric matrix consisting of the cotangent weights only (without the division by Voronoi areas). In other words, it evaluates an "integrated" Laplacian rather than an "averaged" Laplacian when applied to a vector of vertices. The inverse mass matrix appearing in the formula above then applies the appropriate rescaling so that the Laplacian operator can be applied again (i.e., so that the Laplacian value computed at each vertex can be interpreted as a piecewise linear scalar field whose Laplacian can be computed).

After applying the above equation we get a new mesh ***B*** that has the high-frequency details from the part of the surface we want to deform removed.

### Step 2: Deforming the smooth mesh
<img align="left" width="200" src="img/hand_t.png">
<img align="left" width="200" src="img/woody_t.png">
<br clear="both"/>

The computation of the new deformed mesh is done by solving the minimization (similarly to previous step):<br/>
<img align="center" width="200" src="https://i.imgur.com/xv8ZcsA.png">
<!-- $$
\begin{aligned} \min_\textbf{v}& \quad \textbf{v}^T \textbf{L}_\omega \textbf{M}^{-1} \textbf{L}_\omega \textbf{v} \\
 \text{subject to}&
 \quad \textbf{v}_H = t(\textbf{o}_H),
\end{aligned}
$$ -->
where *t*(**O**<sub>*H*</sub>) are the new handle vertex positions after applying the user's transformation. We then call this new mesh ***B'***

### Transferring high-frequency details to the deformed surface
<img align="left" width="200" src="img/hand_bd.png">
<img align="left" width="200" src="img/hand_td.png">
<br clear="both"/>
<img align="left" width="200" src="img/woody_bd.png">
<img align="left" width="200" src="img/woody_td.png">
<br clear="both"/>

The high-frequency details on the original surface are extracted from ***S*** and transferred to ***B'***. First encode the high-frequency details of ***S*** w.r.t ***B***. We define an orthogonal reference frame on every vertex *v* of ***B*** using:

1. The unit vertex normal
2. The normalized projection of one of *v*'s outgoing edges onto the tangent plane defined by the vertex normal. A stable choice is the edge whose projection onto the tangent plane is longest.
3. The cross-product between (1) and (2)

For every vertex *v*, we compute the displacement vector that takes *v* from ***B*** to ***S*** and represent it as a vector in *v*'s reference frame. For every vertex of ***B'*** , we also construct a reference frame using the normal and the same outgoing edge we selected for ***B***. We can now use the displacement vector components computed in the previous paragraph to define transferred displacement vectors in the new reference frames of ***B'*** . Applying the transferred displacements to the vertices of ***B'*** generates the final deformed mesh ***S'*** .

<img align="left" width="200" src="img/hand_f.png">
<img align="left" width="200" src="img/woody_f.png">
<br clear="both"/>
