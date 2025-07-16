<div align=center>   

<img src="https://komarev.com/ghpvc/?username=CPScript&style=flat-square&color=blue" alt="added to profile on 11/23"/>
<img src="https://img.shields.io/github/stars/CPScript?label=Stars" alt="total amount of stars">

</div>

*“Code is like humor. When you have to explain it, it’s bad.”* – **Cory House**

---

<div align="left">     

<div align="center">
  <img src="divider2.png" alt="divider"/>
</div> 

<div align="left">

![image](https://github.com/user-attachments/assets/605d20de-0a1f-403d-8ad3-33bf0f261d03)

<details>
<summary><b>Click to expose; GLSL shader script ^^^^</b></summary>

```
#define BLACK_HOLE_RADIUS 1.0
#define SCHWARZSCHILD_RADIUS 0.4
#define ACCRETION_DISK_INNER 1.0
#define ACCRETION_DISK_OUTER 4.0
#define ACCRETION_DISK_THICKNESS 0.1
#define DISK_TEMPERATURE_SCALE 1.5
#define LENSING_STRENGTH 2.5
#define DOPPLER_STRENGTH 1.2
#define GRAVITATIONAL_REDSHIFT 0.9
#define ROTATION_SPEED 0.2
#define STAR_DENSITY 200.0
#define DUST_DENSITY 0.4

float hash(vec2 p) {
    p = fract(p * vec2(123.45, 678.91));
    p += dot(p, p + 45.32);
    return fract(p.x * p.y);
}

float noise(vec2 p) {
    vec2 i = floor(p);
    vec2 f = fract(p);
    f = f * f * (3.0 - 2.0 * f);

    float a = hash(i);
    float b = hash(i + vec2(1.0, 0.0));
    float c = hash(i + vec2(0.0, 1.0));
    float d = hash(i + vec2(1.0, 1.0));

    return mix(mix(a, b, f.x), mix(c, d, f.x), f.y);
}

vec3 starField(vec2 uv, float time) {
    float stars1 = pow(noise(uv * STAR_DENSITY), 20.0) * 1.0;
    float stars2 = pow(noise(uv * STAR_DENSITY * 0.5 + 30.0), 20.0) * 1.5;
    float stars3 = pow(noise(uv * STAR_DENSITY * 0.25 + 10.0), 20.0) * 2.0;
    
    stars1 *= 0.8 + 0.2 * sin(time * 1.5 + uv.x * 10.0);
    stars2 *= 0.8 + 0.2 * sin(time * 0.7 + uv.y * 12.0);
    stars3 *= 0.8 + 0.2 * cos(time * 1.0 + uv.x * uv.y * 5.0);

    vec3 color1 = vec3(0.8, 0.9, 1.0) * stars1; 
    vec3 color2 = vec3(1.0, 0.9, 0.7) * stars2; 
    vec3 color3 = vec3(1.0, 0.6, 0.5) * stars3; 
    
    return color1 + color2 + color3;
}

vec3 nebulaEffect(vec2 uv, float time) {
    vec3 nebula = vec3(0.0);
    float t = time * 0.05;
    
    float n1 = noise(uv * 1.0 + t);
    float n2 = noise(uv * 2.0 - t * 0.5);
    float n3 = noise(uv * 4.0 + t * 0.2);
    
    float nebulaNoise = pow(n1 * n2 * n3, 3.0) * DUST_DENSITY;
    
    nebula += vec3(0.2, 0.1, 0.3) * nebulaNoise * 2.0; 
    nebula += vec3(0.1, 0.2, 0.4) * nebulaNoise * 1.5;
    nebula += vec3(0.3, 0.1, 0.2) * pow(n3, 4.0) * 0.8;
    
    return nebula;
}

vec3 dopplerShift(vec3 color, float velocity) {
    float doppler = 1.0 + velocity * DOPPLER_STRENGTH;

    return vec3(
        color.r * (velocity < 0.0 ? 1.0/doppler : 1.0),
        color.g,
        color.b * (velocity > 0.0 ? 1.0/doppler : 1.0)
    );
}

vec3 temperatureColor(float temperature) {
    vec3 color = vec3(1.0);
    
    color.r = pow(temperature, 1.5);
    
    color.g = pow(temperature, 2.0) * (1.0 - temperature * 0.5);
    
    color.b = pow(temperature, 3.0) * (1.0 - temperature * 0.8);
    
    color = normalize(color) * pow(temperature, 1.5);
    
    return color;
}

vec2 raytrace(vec2 uv, float radius, float lensStrength) {
    float r = length(uv);
    float theta = atan(uv.y, uv.x);
    
    float bendingFactor = lensStrength * SCHWARZSCHILD_RADIUS / max(r, 0.001);
    float bendingAmount = 1.0 / (1.0 + pow(r / radius, 2.0) * exp(-bendingFactor));
    
    float newRadius = mix(r, radius * radius / r, bendingAmount);
    
    return vec2(cos(theta), sin(theta)) * newRadius;
}

void mainImage(out vec4 fragColor, in vec2 fragCoord) {
    vec2 uv = (fragCoord - 0.5 * iResolution.xy) / iResolution.y;
    
    float time = iTime * 0.5;
    
    vec2 lensedUV = raytrace(uv, BLACK_HOLE_RADIUS, LENSING_STRENGTH);
    
    float r = length(lensedUV);
    float theta = atan(lensedUV.y, lensedUV.x);
    
    float rotatedTheta = theta + time * ROTATION_SPEED;
    vec2 diskUV = vec2(r * cos(rotatedTheta), r * sin(rotatedTheta));
    
    float diskDistance = abs(diskUV.y) / ACCRETION_DISK_THICKNESS;
    float diskRadius = length(diskUV);
    float diskMask = smoothstep(ACCRETION_DISK_INNER, ACCRETION_DISK_INNER + 0.1, diskRadius) *
                     smoothstep(ACCRETION_DISK_OUTER + 0.1, ACCRETION_DISK_OUTER, diskRadius) *
                     smoothstep(1.0, 0.0, diskDistance);
    
    float temperature = mix(0.3, 1.0, smoothstep(ACCRETION_DISK_OUTER, ACCRETION_DISK_INNER, diskRadius)) * DISK_TEMPERATURE_SCALE;
    vec3 diskColor = temperatureColor(temperature);
    
    float velocity = sin(rotatedTheta) * 0.8 * smoothstep(ACCRETION_DISK_OUTER, ACCRETION_DISK_INNER, diskRadius);
    diskColor = dopplerShift(diskColor, velocity);
    
    float redshiftFactor = mix(1.0, GRAVITATIONAL_REDSHIFT, smoothstep(ACCRETION_DISK_OUTER * 0.5, ACCRETION_DISK_INNER, diskRadius));
    diskColor *= redshiftFactor;
    
    float blackHoleMask = 1.0 - smoothstep(SCHWARZSCHILD_RADIUS * 0.9, SCHWARZSCHILD_RADIUS, r);

    vec2 starUV = mix(uv, lensedUV, smoothstep(5.0, 1.0, length(uv)));
    vec3 stars = starField(starUV * 0.5, time);
    
    vec3 nebula = nebulaEffect(starUV * 0.2, time) * 0.3;
    
    float photonRing = smoothstep(SCHWARZSCHILD_RADIUS - 0.03, SCHWARZSCHILD_RADIUS, r) * 
                       smoothstep(SCHWARZSCHILD_RADIUS + 0.03, SCHWARZSCHILD_RADIUS, r);
    vec3 photonRingColor = vec3(1.0, 0.8, 0.6) * 5.0 * photonRing;
    
    float blueShiftGlow = pow(max(0.0, -sin(rotatedTheta)), 4.0) * diskMask * 2.0;
    vec3 blueShiftColor = vec3(0.5, 0.7, 1.0) * blueShiftGlow;
    
    vec3 color = vec3(0.0);
    
    color += (stars + nebula) * (1.0 - blackHoleMask);
    
    color += diskColor * diskMask * 3.0;
    
    color += photonRingColor;
    
    color += blueShiftColor;
    
    color += max(vec3(0.0), color - 1.0) * 0.5;

    color = pow(color, vec3(0.8)); 
    color = (color - 0.1) * 1.1;
    
    fragColor = vec4(max(vec3(0.0), color), 1.0);
}
```
</details>

<details open>
<summary>My Stats</summary>
<br>
            
![GitHub Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=CPScript&layout=compact&theme=blue-green)
<img src="https://stats4github.vercel.app/api/top-langs/?username=CPScript&langs_count=11&hide=html&layout=compact&exclude_repo=Viruses,terminal,Joker,Rosehip-android"><br/>


![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=CPScript&show_icons=true&theme=synthwave)

[![GitHub Streak](https://github-readme-streak-stats.herokuapp.com?user=CPScript&theme=hacker&date_format=M%20j%5B%2C%20Y%5D)](https://git.io/streak-stats)
              
<table><tbody><tr><td><a href="https://octo-ring.com/"><img src="https://octo-ring.com/static/img/widget/top.png" width="99%" alt="Octo Ring logo" align="top"></a><br><a href="https://octo-ring.com/p/CPScript/prev"><img src="https://octo-ring.com/static/img/widget/prev.png" width="33%" alt="previous" align="top" title="previous profile"></a><a href="https://octo-ring.com/p/CPScript/random"><img src="https://octo-ring.com/static/img/widget/random.png" width="33%" alt="random" align="top" title="random profile"></a><a href="https://octo-ring.com/p/CPScript/next"><img src="https://octo-ring.com/static/img/widget/next.png" width="33%" alt="next" align="top" title="next profile"></a><br><a href="https://octo-ring.com/"><img src="https://octo-ring.com/static/img/widget/bottom.png" width="99%" alt="check out other GitHub profiles in the Octo Ring" align="top"></a></td></tr></tbody></table>
    
</details>
