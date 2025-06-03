+++
title = 'A Journey Through Integer Linear Programming Land'
date = 2025-06-02T18:54:46+02:00
weight = 1
[params]
math = true
+++

[Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) is a
powerful modelling tool which lets you efficiently find optimal solutions to
problems described as a linear objective function and linear constraints
on a set of so called _decision variables_, i.e. what you want to optimise.
[Integer Linear Programming](https://en.wikipedia.org/wiki/Linear_programming#Integral_linear_programs)
is just that but applied to problems with integer decision variables rather than
continuous ones. It's usually through (binary) integer values that you are able to
describe conditional statements in your model.

## A first (wrong) model

Ignoring lecture presence constraints, the problem can be modelled as

<!--more-->

$$
\vec{p}^* = \arg\min_{\vec{p}} \sum_t \left( \vec{B} \cdot \vec{\Delta p}^{(t)} + x^{(t)} + y^{(t)} \right)
$$

subject to

$$
\begin{align}
    \sum_i p_i^{(t)} &= 1 & \forall t \\
    p_i^{(t)} &\le \overline O_i^{(t)} & \forall i, t \\
    x^{(t)} &\ge (\vec{F} - M\vec{B}) \cdot \vec{\Delta p}^{(t)} & \forall t \\
    y^{(t)} &\ge (\vec{R} - M (\vec{B} * \vec{F})) \cdot \vec{\Delta p}^{(t)} & \forall t \\
    p_i^{(t)} &\in \{0, 1\} & \forall i, t \\
    x^{(t)}, y^{(t)} &\in \mathbb{N}_+ & \forall t
\end{align}
$$

where $ \vec{\Delta p}^{(t)} = \vec{p}^{(t)} - \vec{p}^{(t-1)} $
and $ \vec{p}^{(0)} $ is given, and $\vec x \cdot \vec y$ represents
the scalar product and $\vec x * \vec y$, the element-wise product
between vectors $\vec x$ and $\vec y$.

In this model, we represent decision variables with lower-case
letters and inputs with upper case ones.
Thus, this model takes as inputs

$\vec{B}$
: The building number $B_i$ associated with each room $i$

$\vec{F}$
: The floor number $F_i$ associated with each room $i$

$\vec{R}$
: The room number $R_i$ associated with each room $i$

$\overline O_i^{(t)}$
: The complement of the occupancy of room $i$ at time $t$ ($0$ if the 
  room is occupied, otherwise $1$)

$M$
: A large integer constant

and outputs the decision variables

$p_i^{(t)}$ 
: representing whether the student should be present in room $i$ at 
  time $t$

$x^{(t)}$ and $y^{(t)}$
: which are auxiliary and not relevant for the interpretation of the 
  solution

The problem can equivalently be described foregoing vector notation
as

$$
p^* =  \arg\min_{p} \sum_t \left( \sum_i {B}_i {\Delta p}^{(t)}_i + x^{(t)} + y^{(t)} \right)
$$

subject to

$$
\begin{align}
    \sum_i p_i^{(t)} &= 1 & \forall t \\
    p_i^{(t)} &\le \overline O_i^{(t)} & \forall i, t \\
    x^{(t)} &\ge \sum_i ({F}_i - M{B}_i) {\Delta p}^{(t)}_i & \forall t \\
    y^{(t)} &\ge \sum_i ({R}_i - M{B}_i{F}_i) {\Delta p}^{(t)}_i & \forall t \\
    p_i^{(t)} &\in \{0, 1\} & \forall i, t \\
    x^{(t)}, y^{(t)} &\in \mathbb{N}_+ & \forall t
\end{align}
$$

Lecture presence constraints could be added simply  by considering
an input variable $L_i^{(t)}$ representing whether the student has
a lecture in room $i$ at time $t$, and then adding that to the
occupancy constraint [^occupancy]

[^occupancy]: The room will be occupied if the student has a lecture
so it is only natural to consider both constraints together.

$$
p_i^{(t)} \le \overline O_i^{(t)} + L_i^{(t)} \quad \forall i, t
$$

### How does it work

The idea behind this model is we penalise a solution when it switches
buildings, floors within a building, or rooms within a floor, but we
only penalise it for one of these components.

Let's expand the constraint on $x^{(t)}$ to show that more clearly

$$
\begin{split}
x^{(t)} &\ge    \sum_i ({F}_i - M{B}_i) {\Delta p}^{(t)}_i \\
        &=      \sum_i {F}_i {\Delta p}^{(t)}_i - M {B}_i {\Delta p}^{(t)}_i \\
        &=      \sum_i {F}_i p_i^{(t)} - F_i p_i^{(t-1)} - M ({B}_i p_i^{(t)} - B_i p_i^{(t-1)})
\end{split}
$$

An important consideration in this model is that the values of
$B_i$ and $F_i$ are _the same for every room in that building or
floor_, respectively.
Another is that $p_i^{(t)}$ is a one-hot vector, meaning only a single
value is $1$ at any time, and all others are $0$, as enforced by the
first constraint.

Now, if we change buildings, ${B}_i p_i^{(t)} \ne B_i p_i^{(t-1)}$
and so we force $x^{(t)}$ to zero, given natural and positive
$x^{(t)}$ and a sufficiently large $M$; if on the other hand we stay
in the same building, we penalise changing floors within the building
by forcing $x^{(t)} \ge F_i {\Delta p}^{(t)}_i$.

### How does it _not_ work

One very clear flaw in this model is that we may end up forcing the
penalisation factors to be zero in the constraints, depending on
how we move (for example, moving from a higher floor to a lower 
floor). That's because ${F}_i p_i^{(t)} - F_i p_i^{(t-1)}$ in
the example we just saw may end up being negative, meaning the
penalisation for changing rooms would be 0 even though we are
indeed changing.

Another more subtle flaw is we're penalising moving arbitrarily.
In _Politecnico_ it is usually the case that, say, building $X$ and
$X+1$ are closer than buildings $X$ and $X+2$, but that's not always 
the case (e.g. building 5 and 9 are closer than 5 and 7). To deal 
with that, we'll need to consider the physical distances between 
buildings and rooms at the university.

## A correct model

We can model physical distances as

$$
p^* = \arg\min_{p} \sum_t {d_{B_x}}^{(t)} + {d_{B_y}}^{(t)} + {d_{F}}^{(t)} + {d_{R_x}}^{(t)} + {d_{R_y}}^{(t)}
$$

subject to

$$
\begin{align}
    \sum_r p^{(r, t)} &= 1 & \forall t \\
    {d_{B_x}}^{(t)} &\ge \pm \vec{B_x} \cdot \vec{{\Delta p}^{(t)}} & \forall t \\
    {d_{B_y}}^{(t)} &\ge \pm \vec{B_y} \cdot \vec{{\Delta p}^{(t)}} & \forall t \\
    {d_F}^{(t)} &\ge \vec{F} \cdot \vec{{\Delta p}^{(t)}}  - {a_F}^{(t)} & \forall t \\
    {d_{R_x}}^{(t)} &\ge \vec{R_x} \cdot \vec{{\Delta p}^{(t)}}  - {a_R}^{(t)} & \forall t \\
    {d_{R_y}}^{(t)} &\ge \vec{R_y} \cdot \vec{{\Delta p}^{(t)}}  - {a_R}^{(t)} & \forall t \\
    {a_F}^{(t)} &\ge \pm  M (\vec{B_x} * \vec{B_y}) \cdot \vec{{\Delta p}^{(t)}} & \forall t \\
    {a_{R}}^{(t)} &\ge \pm  M (\vec{F} * \vec{B_x} * \vec{B_y}) \cdot \vec{{\Delta p}^{(t)}} & \forall t \\
    p^{(r, t)} &\le \overline O^{(r, t)}  + L^{(r, t)} & \forall r, t \\
    p^{(r, t)} &\in \{0, 1\} & \forall r, t \\
\end{align}
$$

with
$
{d_{B_x}}^{(t)},
{d_{B_y}}^{(t)},
{d_F}^{(t)},
{d_{R_x}}^{(t)},
{d_{R_y}}^{(t)},
{a_F}^{(t)},
{a_{R}}^{(t)}
\in  \mathbb{N}_+ \; \forall t
$
.

I changed the notation slightly so only $B$, $F$ or $R$ remained as
subscripts; the additional superscript $^{(r)}$ represents the room and
has the same function as the previous $i$ subscript.

### How it _does_ work

Since computing the Euclidean distance is a non-linear operation, we
use the Manhattan distance[^manhattan] by adding distances in
$x$ and $y$ for buildings ($B_x, B_y$) and rooms ($R_x, R_y$).
We keep the same index-based distance for floors since it's reasonable
to expect the 1st floor to be twice as close to the 2nd as to the 3rd.

[^manhattan]: One could argue the Manhattan distance is best at any
rate since it better captures how people move while walking; it's
the city-block distance after all!

We introduce new auxiliary decision variables $a$ which act as the
modulus operand over their arguments in practice; with $a_R$ for 
instance

$$ {a_R^{(t)}}^* = \left| M (\vec{F} * \vec{B_x} * \vec{B_y}) \cdot \vec{\Delta {p^{(t)}}^*} \right| \quad \forall t $$

And the same is true for $a_F$, since their constraints specify they
must be greater than 0 and _greater than the positive and negative_
values of the expression. 

The idea behind the objective components remains the same: we disable
those which are not relevant, so we consider only building cost when
changing buildings, only floor cost when changing floors in the same 
building, and only room cost when changing rooms in the same floor of
the same building.

Alright, so we're done then!

## Not so fast

Even though the model may be correct, we still need to somehow
**gather the data!** Not only about the lecture schedules, but also
about the position of the buildings _and_ the rooms inside the
buildings at _Politecnico_.

But that'll be for another post.
