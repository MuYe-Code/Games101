# 投影变换

## 3D Transformation(Model Transformation)

### Scale(缩放)

$x坐标缩放到s_x倍，y坐标缩放到s_y倍，z坐标缩放到s_z倍$
$$
S(s_x,x_y,s_z)=\left[
\matrix{
s_x & 0 & 0 & 0\\
0 & s_y & 0 & 0\\
0 & 0 & s_z & 0\\
0 & 0 & 0 & 1\\
}
\right]
$$

### Translation(平移)

$将点从(t_x,t_y,t_z)位移到点(0,0,0)，即将图形移动到原点$
$$
T(t_x,t_y,t_z)=\left[
\matrix{
1 & 0 & 0 & t_x\\
0 & 1 & 0 & t_y\\
0 & 0 & 1 & t_z\\
0 & 0 & 0 & 1\\
}\right]
$$

### Rotation(旋转)

#### Rotation around x-,y-,or z-axis

$逆时针旋转\alpha度，R_{xyz}(\alpha,\beta,\gamma)=R_x(\alpha)R_y(\beta)R_z(\gamma)$
$$
R_x(\alpha)=\left[
\matrix{
1 & 0 & 0 & 0\\
0 & cos\alpha & -sin\alpha & 0\\
0 & sin\alpha & cos\alpha & 0\\
0 & 0 & 0 & 1
}
\right]
$$

$$
R_y(\alpha)=\left[
\matrix{
cos\alpha & 0 & sin\alpha & 0\\
0 & 1 & 0 & 0\\
-sin\alpha & 0 & cos\alpha & 0\\
0 & 0 & 0 & 1
}
\right]
$$

$$
R_z(\alpha)=\left[
\matrix{
cos\alpha & -sin\alpha & 0 & 0\\
sin\alpha & cos\alpha & 0 & 0\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
}
\right]
$$

#### Rotation by angle $\alpha$ around axis n

$$
R(\textbf{n},\alpha)=cos(\alpha)\textbf{I}+(1-cos(\alpha))\textbf{nn}^T+sin(\alpha)\left[
\matrix{
0 & -n_z & n_y\\
n_z & 0 & -n_x\\
-n_y & n_x & 0
}
\right]
$$

```c++
//物体绕Z轴逆时针旋转角度angle
Eigen::Matrix4f get_model_matrix(float angle)
{
    Eigen::Matrix4f rotation;
    angle = angle * MY_PI / 180.f;
    //获取旋转矩阵
    rotation << cos(angle), 0, sin(angle), 0,
                0, 1, 0, 0,
                -sin(angle), 0, cos(angle), 0,
                0, 0, 0, 1;
	//获取放缩矩阵（放大到2.5倍）
    Eigen::Matrix4f scale;
    scale << 2.5, 0, 0, 0,
              0, 2.5, 0, 0,
              0, 0, 2.5, 0,
              0, 0, 0, 1;
	//获取平移矩阵
    Eigen::Matrix4f translate;
    translate << 1, 0, 0, 0,
            0, 1, 0, 0,
            0, 0, 1, 0,
            0, 0, 0, 1;

    return translate * rotation * scale;
}
```



## Viewing transformation

### View/Camera Transformation

#### Transform the camera by $M_{view}$: locate at the origin,  look at -z

$M_{view}=R_{view}T_{view}$

1. Translate the camera to origin:

$$
T_{view}=\left[
\matrix{
1 & 0 & 0 & -x_e\\
0 &1 & 0 & -y_e\\
0 & 0 & 1 & -z_e\\
0 & 0 & 0 & 1
}
\right]
$$

2. Rotate g to -Z, t to Y, (g$\times$t) to X

   Consider its inverse rotation: X to (g$\times$t), Y to t, Z to -g

   正则矩阵的逆矩阵等于转置矩阵
   $$
   R_{view}^{-1}=\left[
   \matrix{
   x_{g\times t} & x_t & x_{-g} & 0\\
   y_{g\times t} & y_t & y_{-g} & 0\\
   z_{g\times t} & z_t & z_{-g} & 0\\
   0 & 0 & 0 & 1\\
   }
   \right]
   =>
   R_{view}=\left[
   \matrix{
   x_{g\times t} & y_{g\times t} & z_{g\times t} & 0\\
   x_t & y_t & z_t & 0\\
   x_{-g} & y_{-g} & z_{-g} & 0\\
   0 & 0 & 0 &1
   }
   \right]
   $$

```c++
//将相机移动到原点并让其对准正确方向
Eigen::Matrix4f get_view_matrix(Eigen::Vector3f eye_pos)
{
    Eigen::Matrix4f view = Eigen::Matrix4f::Identity();

    Eigen::Matrix4f translate;
    translate << 1,0,0,-eye_pos[0],
                 0,1,0,-eye_pos[1],
                 0,0,1,-eye_pos[2],
                 0,0,0,1;

    view = translate*view;

    return view;
}
```



## Projection Transformation

### Orthographic Projection

先平移，再缩放(矩阵是右结合的)
$$
M_{ortho}=\left[
\matrix{
\frac{2}{r-l} & 0 & 0 & 0\\
0 & \frac{2}{t-b} & 0 & 0\\
0 & 0 & \frac{2}{n-f} & 0\\
0 & 0 & 0 & 1
}
\right]
\left[
\matrix{
1 & 0 & 0 & -\frac{r+l}{2}\\
0 & 1 & 0 & -\frac{t+b}{2}\\
0 & 0 & 1 & -\frac{n+f}{2}\\
0 & 0 & 0 & 1
}
\right]
$$


### Perspective Projection

透视投影到近面
$$
M_{persp->ortho}=\left[
\matrix{
n & 0 & 0 & 0\\
0 & n & 0 & 0\\
0 & 0 & n+f & -nf\\
0 & 0 & 1 & 0\\
}
\right]
$$
$M_{projection}=M_{ortho}M_{persp->ortho}$

```c++
//获取投影矩阵
//eye_fov是可视角度，zNear是近面的坐标，zFar是远面的坐标
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio, float zNear, float zFar)
{
    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();
    Eigen::Matrix4f m = Eigen::Matrix4f::Identity();
    //透视转正交
    m << zNear, 0, 0, 0, 0, zNear, 0, 0, 0, 0, zFar+zNear, -zFar*zNear, 0, 0, 1, 0;
    float half_r = MY_PI*(eye_fov/2)/180.0;
    float top=atan(half_r)*zNear, bottom=-top;
    float right=top*aspect_ratio, left=-right;
    Eigen::Matrix4f scale, transpos;
    //缩放到标准大小
    scale << 2/(right-left), 0, 0, 0, 0, 2/(top-bottom), 0, 0, 0, 0, 2/(zNear-zFar), 0, 0, 0, 0, 1;
    //将物体移动到原点
    transpos << 1, 0, 0, -(right+left)/2, 0, 1, 0, -(top+bottom)/2, 0, 0, 1, -(zNear+zFar)/2, 0, 0, 0, 1;
    //计算透视投影矩阵
    projection=scale*transpos*m;
    return projection;
}
```

