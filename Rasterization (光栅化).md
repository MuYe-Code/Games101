# Rasterization (光栅化) 

## Triangle

```c++
static bool insideTriangle(float x, float y, const Vector3f* _v)
{   
    // TODO : Implement this function to check if the point (x, y) is inside the triangle represented by _v[0], _v[1], _v[2]
    //构建三条边的向量
    Eigen::Vector3f ab=_v[1]-_v[0];
    Eigen::Vector3f bc=_v[2]-_v[1];
    Eigen::Vector3f ca=_v[0]-_v[2];
    
    Eigen::Vector3f point;
    point << x, y, 1;
    Eigen::Vector3f ap=point-_v[0];
    Eigen::Vector3f bp=point-_v[1];
    Eigen::Vector3f cp=point-_v[2];
    
    float z1=ab.x()*ap.y()-ab.y()*ap.x();
    float z2=bc.x()*bp.y()-bc.y()*bp.x();
    float z3=ca.x()*cp.y()-ca.y()*cp.x();
    //符号一致说明在三角形内
    if(z1>0 && z2>0 && z3>0 || z1<0 && z2<0 && z3<0) return true;
    else return false;
}
```

## Antialiasing

Rasterization = Sample 2D Positions

Sample Artifaxts: Jaggis, Moire, Wagon wheel effect

采样频率过低，但信号变化频率太高，导致了图片缺陷
解决方法：Filter then sample(过滤高频信息)



频域的滤波（乘法）等于时域的卷积

Filtering = Convolution



MSAA: Approximate the effect of the 1-pixel box filter by sampling multiple locations within a pixel and averaging their values.(超采样)

```c++
//Screen space rasterization
void rst::rasterizer::rasterize_triangle(const Triangle& t) {
    auto v = t.toVector4();
 
    // TODO : Find out the bounding box of current triangle.
    float min_x=std::min(v[0].x(),std::min(v[1].x(),v[2].x()));
    float min_y=std::min(v[0].y(),std::min(v[1].y(),v[2].y()));
    float max_x=std::max(v[0].x(),std::max(v[1].x(),v[2].x()));
    float max_y=std::max(v[0].y(),std::max(v[1].y(),v[2].y()));
    // iterate through the pixel and find if the current pixel is inside the triangle
    
    bool msaa=true;
    if(msaa){
        std::vector<Eigen::Vector2f> pos={
            //一个像素点进行四次采样
            {0.25,0.25},{0.25,0.75},
            {0.25,0.75},{0.75,0.75}
        };
        for(int x=min_x;x<=max_x;x++){
            for(int y=min_y;y<=max_y;y++){
                //count记录有几次采样位与三角形内部
                int count=0;
                //mindepth用于记录该像素点的深度信息
                float mindepth=FLT_MAX;
                //依次处理四个采样点
                for(int i=0;i<4;i++){
                    if(insideTriangle(x+pos[i][0],y+pos[i][1],t.v)){
                        count++;
                        //对三角形内部的像素点线性插值
                        auto[alpha, beta, gamma] = computeBarycentric2D(x+0.5f, y+0.5f, t.v);
                        float w_reciprocal = 1.0/(alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                        float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                        //修正投影变换的误差
                        z_interpolated *= w_reciprocal;
						
                        //深度信息取距离原点更近的
                        mindepth=std::min(mindepth,z_interpolated);
                    }
                }
                //如果有该像素至少一个采样点位于三角形内，则取颜色的加权平均
                if(count!=0){
                    if(depth_buf[get_index(x,y)]>mindepth){
                        Eigen::Vector3f color=t.getColor()*count/4.0;
                        Eigen::Vector3f point;
                        point<<(float)x, (float)y, mindepth;
                        //更新深度信息及颜色
                        depth_buf[get_index(x,y)]=mindepth;
                        set_pixel(point,color);
                    }
                }
            }
        }

    }
    else {
        for(int x=min_x; x<=max_x; x++){
        for(int y=min_y; y<=max_y; y++){
            if(insideTriangle(x+0.5f,y+0.5f,t.v)){
                auto[alpha, beta, gamma] = computeBarycentric2D(x+0.5f, y+0.5f, t.v);

                float w_reciprocal = 1.0/(alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                z_interpolated *= w_reciprocal;

                if(z_interpolated<depth_buf[get_index(x,y)]){
                    Eigen::Vector3f point;
                    point << (float)x, (float)y, z_interpolated;
                    depth_buf[get_index(x,y)]=z_interpolated;
                    set_pixel(point,t.getColor());
                }
            }
        }
    }
    }
    // If so, use the following code to get the interpolated z value.
    //auto[alpha, beta, gamma] = computeBarycentric2D(x, y, t.v);
    //float w_reciprocal = 1.0/(alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
    //float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
    //z_interpolated *= w_reciprocal;

    // TODO : set the current pixel (use the set_pixel function) to the color of the triangle (use getColor function) if it should be painted.
}
```

