# 3D Maths

## Quaternions

### Pitfalls of matrix rotations

- requires 9 floats to represent a rotation of 3 degrees of freedom
- matrix-vector product is expensive
- cannot easily interpolate between two rotations, which is useful in, e.g., animation

### Definition

$$\text{q} = [q_x\;q_y\;q_z\;q_w]$$

unit quaternions are 3D rotations:

$$\text{q} = [\textbf{v}\sin(\theta/2)\;\cos(\theta/2)] = [\textbf{q}_v\;q_S]$$

where $\text{v}$ is the (unit) axis of rotation and $\theta$ is the angle of rotation, following right-hand rule.

### Operations

#### Multiplication

$pq$ represents applying corresponding rotations $p$ and $q$ in sequence. The multiplication is defined as follows:

$$ \text{pq} = [(p_S\textbf{q}_V + q_S\textbf{p}_V + \textbf{p}_V\times \textbf{q}_V) \;\; (p_Sq_S - \textbf{p}_V\cdot \textbf{q}_V)]$$

#### Conjugate and Inverse

Zero rotation is defined as: $\text{q}_0 = [0\;0\;0\;1]$

conjugate of $\text{q}$ is $\text{q}^* = [-\textbf{q}_V\;q_S]$

and the inverse of $\text{q}$ is $\text{q}^{-1} = \frac{\text{q}^*}{|\text{q}|^2} = \text{q}^*$ , since $|\text{q}| = 1$

conjugate and inverse of a product use the same rules as in matrix algebra.

#### Rotating vectors

1. Rewrite the vector in quaternion form: $\text{v} = [v_x\;v_y\;v_z\;0]$
2. Rotate the vector: $\text{v}' = \text{q}\text{v}\text{q}^{-1}$
3. Extract $\textbf{v}'$ from the quaternion $\text{v}'$, which gives the result

### Interpolation

To be examined later...

### Comparison of rotational representations

- Euler angles
  - Cannot be interpolated when the axes are arbitrary
  - prone to gimbal lock: rotating one axis to 90 degrees makes it align with another, losing a degree of freedom
  - order of rotations matters