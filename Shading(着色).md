# Shading(着色)

Specular highlights + Diffuse reflection + Ambient light

## Diffuse Reflection

Surface color is the same for all viewing directions.
$$
L_d=k_d(I/r^2)max(0,\pmb{n·l})
$$
$L_d:\text{diffusely reflected light}$

$k_d:\text{diffuse coefficient(color)}$

$I/r^2:\text{energy arrived at the shading point}$

$max(0,\pmb{n·l}):\text{energy received by the shading piont}$

## Specular Term(Blinn-Phong)

Intensity depends on view direction.

V close to mirror direction <=> half vector near normal: Measure "near" by dot product of unit vectors.
$$
\pmb{h}=bisector(\pmb{v,l})=\frac{\pmb{v,l}}{||\pmb{v+l}||}
$$

$$
L_s=k_s(I/r^2)max(0,cos\alpha)^p=k_s(I/r^2)max(0,\pmb{n·h})^p
$$

n is normal direction, h is half vector.

If $cos\alpha$ is negative, it means the direction of light is at the back of the surface.

## Ambient Term

Shading that does not depend on anything.
$$
L_a=k_aI_a
$$
$L_a:\text{reflected ambient light}\\$

$k_a:\text{ambient coeffient}$


$$
L=L_a+L_s+L_d=k_aI_a+k_d(I/r^2)max(0,\pmb{n·l})+k_s(I/r^2)max(0,\pmb{n·h})^p
$$

## Shade each vertex -- Gouraud shading

Interpolate colors from vertices across triangle.

## Shade each pixel -- Phong shading

Interpolate normal vectors across each triangle.

Compute full shading model at each pixel.

Not the Blinn-Phong Reflectance Model!



# Texture Mapping

#### Virsualization of Texture Coordinates

Each triangle vertex is assigned a texture coordinates(u,v)

#### Interpolation Across Triangles -- Barycentric Coordinates

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427185412930.png)

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427185516101.png)

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427185622097.png)
$$
\pmb u\times \pmb v=\left |\begin{array}{cccc}
\pmb i & \pmb j & \pmb k\\
u_1 & u_2 & u_3\\
v_1 & v_2 & v_3
\end{array}\right|
$$
For the cross product of 2D matrix, the result is $x_1y_2-x_2y_1$

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427191402396.png)

### Applying Textures

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427192159259.png)

Different Pixels -> Different_Sized Footprints

#### Mipmap

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427193305749.png)

Get two parameters from level D and Level D+1, the get the final result by interpolation.

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427193508369.png)

Mipmap Limitations -- Overblur

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427193706416.png)

![image-20220427193812044](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427193812044.png)

## Applications of Textures

Textures can affect shading.

##### Bump Mapping -- change the normal direction of vetices.

![](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427205714697.png)

![image-20220427205840587](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427205840587.png)

![image-20220427205902136](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427205902136.png)

##### Displacement Mapping -- move the vertices

![image-20220427210659355](C:\Users\Masker\AppData\Roaming\Typora\typora-user-images\image-20220427210659355.png)





```C
class Texture{
private:
    cv::Mat image_data;

public:
    Texture(const std::string& name)
    {
        image_data = cv::imread(name);
        cv::cvtColor(image_data, image_data, cv::COLOR_RGB2BGR);
        width = image_data.cols;
        height = image_data.rows;
    }

    int width, height;

    Eigen::Vector3f getColor(float u, float v)
    {
        if(u<0) u=0;
        if(u>1) u=1;
        if(v<0) v=0;
        if(v>1) v=1;
        auto u_img = u * width;
        auto v_img = (1 - v) * height;
        auto color = image_data.at<cv::Vec3b>(v_img, u_img);
        return Eigen::Vector3f(color[0], color[1], color[2]);
    }

};

struct fragment_shader_payload
{
    fragment_shader_payload()
    {
        texture = nullptr;
    }

    fragment_shader_payload(const Eigen::Vector3f& col, const Eigen::Vector3f& nor,const Eigen::Vector2f& tc, Texture* tex) :
         color(col), normal(nor), tex_coords(tc), texture(tex) {}


    Eigen::Vector3f view_pos;
    Eigen::Vector3f color;
    Eigen::Vector3f normal;
    Eigen::Vector2f tex_coords;
    Texture* texture;
};

struct vertex_shader_payload
{
    Eigen::Vector3f position;
};


Eigen::Vector3f texture_fragment_shader(const fragment_shader_payload& payload)
{
    Eigen::Vector3f return_color = {0, 0, 0};
    //获取纹理颜色
    if (payload.texture)
    {
        // TODO: Get the texture value at the texture coordinates of the current fragment
        return_color=payload.texture->getColor(payload.tex_coords.x(),payload.tex_coords.y());

    }
    Eigen::Vector3f texture_color;
    texture_color << return_color.x(), return_color.y(), return_color.z();
	
    //三个系数
    Eigen::Vector3f ka = Eigen::Vector3f(0.005, 0.005, 0.005);
    Eigen::Vector3f kd = texture_color / 255.f;
    Eigen::Vector3f ks = Eigen::Vector3f(0.7937, 0.7937, 0.7937);

    auto l1 = light{{20, 20, 20}, {500, 500, 500}};
    auto l2 = light{{-20, 20, 0}, {500, 500, 500}};

    std::vector<light> lights = {l1, l2};
    Eigen::Vector3f amb_light_intensity{10, 10, 10};
    Eigen::Vector3f eye_pos{0, 0, 10};

    float p = 150;

    Eigen::Vector3f color = texture_color;
    Eigen::Vector3f point = payload.view_pos;
    Eigen::Vector3f normal = payload.normal;

    Eigen::Vector3f result_color = {0, 0, 0};

    for (auto& light : lights)
    {
        // TODO: For each light source in the code, calculate what the *ambient*, *diffuse*, and *specular* 
        // components are. Then, accumulate that result on the *result_color* object.
        Eigen::Vector3f light_dir = (light.position-point).normalized();
        Eigen::Vector3f view_dir = (eye_pos-point).normalized();
        Eigen::Vector3f half_vector = (light_dir+view_dir).normalized();
		//环境光
        float r2=(light.position-point).dot(light.position-point);
        Eigen::Vector3f La=ka.cwiseProduct(amb_light_intensity);
		//漫反射
        Eigen::Vector3f Ld=kd.cwiseProduct(light.intensity/r2);
        Ld *= std::max(0.0f,normal.normalized().dot(light_dir));
		//高光
        Eigen::Vector3f Ls=ks.cwiseProduct(light.intensity/r2);
        Ls*=std::pow(std::max(0.0f,normal.normalized().dot(half_vector)),p);

        result_color+=(La+Ld+Ls);
    }

    return result_color * 255.f;
}
```

```C
Eigen::Vector3f phong_fragment_shader(const fragment_shader_payload& payload)
{
    Eigen::Vector3f ka = Eigen::Vector3f(0.005, 0.005, 0.005);
    //模型基底颜色
    Eigen::Vector3f kd = payload.color;
    Eigen::Vector3f ks = Eigen::Vector3f(0.7937, 0.7937, 0.7937);

    auto l1 = light{{20, 20, 20}, {500, 500, 500}};
    auto l2 = light{{-20, 20, 0}, {500, 500, 500}};

    std::vector<light> lights = {l1, l2};
    Eigen::Vector3f amb_light_intensity{10, 10, 10};
    Eigen::Vector3f eye_pos{0, 0, 10};

    float p = 150;

    Eigen::Vector3f color = payload.color;
    Eigen::Vector3f point = payload.view_pos;
    Eigen::Vector3f normal = payload.normal;

    Eigen::Vector3f result_color = {0, 0, 0};
    for (auto& light : lights)
    {
        // TODO: For each light source in the code, calculate what the *ambient*, *diffuse*, and *specular* 
        // components are. Then, accumulate that result on the *result_color* object.

        Eigen::Vector3f light_dir = (light.position-point).normalized();
        Eigen::Vector3f view_dir = (eye_pos-point).normalized();
        Eigen::Vector3f half_vector = (light_dir+view_dir).normalized();

        float r2=(light.position-point).dot(light.position-point);
        Eigen::Vector3f La=ka.cwiseProduct(amb_light_intensity);

        Eigen::Vector3f Ld=kd.cwiseProduct(light.intensity/r2);
        Ld *= std::max(0.0f,normal.normalized().dot(light_dir));

        Eigen::Vector3f Ls=ks.cwiseProduct(light.intensity/r2);
        Ls*=std::pow(std::max(0.0f,normal.normalized().dot(half_vector)),p);

        result_color+=(La+Ld+Ls);
    }

    return result_color * 255.f;
}
```

```C
Eigen::Vector3f normal_fragment_shader(const fragment_shader_payload& payload)
{
    //根据法线方向给予颜色
    Eigen::Vector3f return_color = (payload.normal.head<3>().normalized() + Eigen::Vector3f(1.0f, 1.0f, 1.0f)) / 2.f;
    Eigen::Vector3f result;
    result << return_color.x() * 255, return_color.y() * 255, return_color.z() * 255;
    return result;
}
```

```C
Eigen::Vector3f bump_fragment_shader(const fragment_shader_payload& payload)
{
    
    Eigen::Vector3f ka = Eigen::Vector3f(0.005, 0.005, 0.005);
    Eigen::Vector3f kd = payload.color;
    Eigen::Vector3f ks = Eigen::Vector3f(0.7937, 0.7937, 0.7937);

    auto l1 = light{{20, 20, 20}, {500, 500, 500}};
    auto l2 = light{{-20, 20, 0}, {500, 500, 500}};

    std::vector<light> lights = {l1, l2};
    Eigen::Vector3f amb_light_intensity{10, 10, 10};
    Eigen::Vector3f eye_pos{0, 0, 10};

    float p = 150;

    Eigen::Vector3f color = payload.color; 
    Eigen::Vector3f point = payload.view_pos;
    Eigen::Vector3f normal = payload.normal;


    float kh = 0.2, kn = 0.1;

    // TODO: Implement bump mapping here
    // Let n = normal = (x, y, z)
    // Vector t = (x*y/sqrt(x*x+z*z),sqrt(x*x+z*z),z*y/sqrt(x*x+z*z))
    // Vector b = n cross product t
    // Matrix TBN = [t b n]
    // dU = kh * kn * (h(u+1/w,v)-h(u,v))
    // dV = kh * kn * (h(u,v+1/h)-h(u,v))
    // Vector ln = (-dU, -dV, 1)
    // Normal n = normalize(TBN * ln)
    float x=normal.x();
    float y=normal.y();
    float z=normal.z();

    Eigen::Vector3f t = Eigen::Vector3f(x*y/sqrt(x*x+z*z),sqrt(x*x+z*z),z*y/sqrt(x*x+z*z));
    Eigen::Vector3f b=normal.cross(t);

    Eigen::Matrix3f TBN;
    TBN<<
        t.x(),b.x(),x,
        t.y(),b.y(),y,
        t.z(),b.z(),z;

    float u=payload.tex_coords.x();
    float v=payload.tex_coords.y();
    float w=payload.texture->width;
    float h=payload.texture->height;

    float dU = kh * kn * (payload.texture->getColor(u+1.0f/w,v).norm()-payload.texture->getColor(u,v).norm());
    float dV = kh * kn * (payload.texture->getColor(u,v+1.0f/h).norm()-payload.texture->getColor(u,v).norm());

    Eigen::Vector3f ln=Vector3f(-dU,-dV,1.0f);

    point += (kn*normal*payload.texture->getColor(u,v).norm());

    normal=(TBN*ln).normalized();

    Eigen::Vector3f result_color = {0, 0, 0};

    result_color=normal.normalized();

    return result_color * 255.f;
}
```

```c++
Eigen::Vector3f displacement_fragment_shader(const fragment_shader_payload& payload)
{
    
    Eigen::Vector3f ka = Eigen::Vector3f(0.005, 0.005, 0.005);
    Eigen::Vector3f kd = payload.color;
    Eigen::Vector3f ks = Eigen::Vector3f(0.7937, 0.7937, 0.7937);

    auto l1 = light{{20, 20, 20}, {500, 500, 500}};
    auto l2 = light{{-20, 20, 0}, {500, 500, 500}};

    std::vector<light> lights = {l1, l2};
    Eigen::Vector3f amb_light_intensity{10, 10, 10};
    Eigen::Vector3f eye_pos{0, 0, 10};

    float p = 150;

    Eigen::Vector3f color = payload.color; 
    Eigen::Vector3f point = payload.view_pos;
    Eigen::Vector3f normal = payload.normal;

    float kh = 0.2, kn = 0.1;
    
    // TODO: Implement displacement mapping here
    // Let n = normal = (x, y, z)
    // Vector t = (x*y/sqrt(x*x+z*z),sqrt(x*x+z*z),z*y/sqrt(x*x+z*z))
    // Vector b = n cross product t
    // Matrix TBN = [t b n]
    // dU = kh * kn * (h(u+1/w,v)-h(u,v))
    // dV = kh * kn * (h(u,v+1/h)-h(u,v))
    // Vector ln = (-dU, -dV, 1)
    // Position p = p + kn * n * h(u,v)
    // Normal n = normalize(TBN * ln)


    float x=normal.x();
    float y=normal.y();
    float z=normal.z();

    Eigen::Vector3f t = Eigen::Vector3f(x*y/sqrt(x*x+z*z),sqrt(x*x+z*z),z*y/sqrt(x*x+z*z));
    Eigen::Vector3f b=normal.cross(t);

    Eigen::Matrix3f TBN;
    TBN<<
        t.x(),b.x(),x,
        t.y(),b.y(),y,
        t.z(),b.z(),z;

    float u=payload.tex_coords.x();
    float v=payload.tex_coords.y();
    float w=payload.texture->width;
    float h=payload.texture->height;

    float dU = kh * kn * (payload.texture->getColor(u+1.0f/w,v).norm()-payload.texture->getColor(u,v).norm());
    float dV = kh * kn * (payload.texture->getColor(u,v+1.0f/h).norm()-payload.texture->getColor(u,v).norm());

    Eigen::Vector3f ln=Vector3f(-dU,-dV,1.0f);

    point += (kn*normal*payload.texture->getColor(u,v).norm());

    normal=(TBN*ln).normalized();

    Eigen::Vector3f result_color = {0, 0, 0};


    for (auto& light : lights)
    {
        // TODO: For each light source in the code, calculate what the *ambient*, *diffuse*, and *specular* 
        // components are. Then, accumulate that result on the *result_color* object.

        Eigen::Vector3f light_dir = (light.position-point).normalized();
        Eigen::Vector3f view_dir = (eye_pos-point).normalized();
        Eigen::Vector3f half_vector = (light_dir+view_dir).normalized();

        float r2=(light.position-point).dot(light.position-point);
        Eigen::Vector3f La=ka.cwiseProduct(amb_light_intensity);

        Eigen::Vector3f Ld=kd.cwiseProduct(light.intensity/r2);
        Ld *= std::max(0.0f,normal.normalized().dot(light_dir));

        Eigen::Vector3f Ls=ks.cwiseProduct(light.intensity/r2);
        Ls*=std::pow(std::max(0.0f,normal.normalized().dot(half_vector)),p);

        result_color+=(La+Ld+Ls);
    }

    return result_color * 255.f;
}
```

