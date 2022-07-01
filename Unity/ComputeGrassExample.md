# Credits

Code by Przemyslaw_Zaworski from https://forum.unity.com/threads/feedback-wanted-mesh-compute-shader-access.1096531/

# C#
```csharp
    using Unity.Collections;
    using UnityEngine;
    using UnityEngine.Rendering;
     
    [RequireComponent(typeof(MeshFilter))]
    [RequireComponent(typeof(MeshRenderer))]
    public class ProceduralGrass : MonoBehaviour
    {
        public ComputeShader ProceduralGrassCS;
        public Shader ProceduralGrassPS;  
        public Texture2D GrassTexture;
        public int TriangleCount = 2000000;
     
        GraphicsBuffer _VertexBuffer;
        GraphicsBuffer _NormalBuffer;    
        GraphicsBuffer _TexcoordBuffer;
        Material _Material;
        Mesh _Mesh;
     
        void Update()
        {
            if (_Mesh && _Mesh.vertexCount != TriangleCount * 3)
            {
                Release();
            }
            if (_Mesh == null)
            {
                _Mesh = new Mesh();
                _Mesh.name = "ProceduralGrass";
                _Mesh.vertexBufferTarget |= GraphicsBuffer.Target.Raw;
                VertexAttributeDescriptor[] attributes = new []
                {
                    new VertexAttributeDescriptor(VertexAttribute.Position, stream:0),
                    new VertexAttributeDescriptor(VertexAttribute.Normal, stream:1),
                    new VertexAttributeDescriptor(VertexAttribute.TexCoord0, stream:2)
                };
                _Mesh.SetVertexBufferParams(TriangleCount * 3, attributes);        
                _Mesh.SetIndexBufferParams(TriangleCount * 3, IndexFormat.UInt32);
                NativeArray<int> indexBuffer = new NativeArray<int>(TriangleCount * 3, Allocator.Temp);
                for (int i = 0; i < TriangleCount * 3; ++i) indexBuffer[i] = i;
                _Mesh.SetIndexBufferData(indexBuffer, 0, 0, indexBuffer.Length, MeshUpdateFlags.DontRecalculateBounds | MeshUpdateFlags.DontValidateIndices);
                indexBuffer.Dispose();
                SubMeshDescriptor submesh = new SubMeshDescriptor(0, TriangleCount * 3, MeshTopology.Triangles);
                submesh.bounds = new Bounds(Vector3.zero, new Vector3(2000, 2, 2000));
                _Mesh.SetSubMesh(0, submesh);
                _Mesh.bounds = submesh.bounds;
                GetComponent<MeshFilter>().sharedMesh = _Mesh;
                _Material = new Material(ProceduralGrassPS);
                _Material.mainTexture = GrassTexture;
                GetComponent<MeshRenderer>().sharedMaterial = _Material;
            }
            _VertexBuffer ??= _Mesh.GetVertexBuffer(0);
            _NormalBuffer ??= _Mesh.GetVertexBuffer(1);
            _TexcoordBuffer ??= _Mesh.GetVertexBuffer(2);
            ProceduralGrassCS.SetInt("_TriangleCount", TriangleCount);
            ProceduralGrassCS.SetBuffer(0, "_VertexBuffer", _VertexBuffer);
            ProceduralGrassCS.SetBuffer(0, "_TexcoordBuffer", _TexcoordBuffer);
            ProceduralGrassCS.SetBuffer(0, "_NormalBuffer", _NormalBuffer);
            ProceduralGrassCS.SetFloat("_Time", Time.time);
            ProceduralGrassCS.Dispatch(0, (TriangleCount + 64 - 1) / 64, 1, 1);
        }
     
        void Release()
        {
            Destroy(_Material);
            Destroy(_Mesh);
            _Mesh = null;
            _VertexBuffer?.Dispose();
            _VertexBuffer = null;
            _TexcoordBuffer?.Dispose();
            _TexcoordBuffer = null;
            _NormalBuffer?.Dispose();
            _NormalBuffer = null;
        }
     
        void OnDestroy()
        {
            Release();
        }  
    }
```

# Compute

```csharp
    #pragma kernel CSMain
     
    RWByteAddressBuffer _VertexBuffer;
    RWByteAddressBuffer _NormalBuffer;
    RWByteAddressBuffer _TexcoordBuffer;
    int _TriangleCount;
    float _Time;
     
    struct Vertex
    {
        float3 position;
        float2 texcoord;
    };
     
    void Store2 (RWByteAddressBuffer buffer, int index, float2 v)
    {
        uint2 data = asuint(v);
        buffer.Store2((index*3)<<2, data);
    }
     
    void Store3 (RWByteAddressBuffer buffer, int index, float3 v)
    {
        uint3 data = asuint(v);
        buffer.Store3((index*3) << 2, data);
    }
     
    float Mod (float x, float y)
    {
        return x - y * floor(x/y);
    }
     
    float3x3 RotationY (float y)
    {
        return float3x3(cos(y),0.0,-sin(y), 0.0,1.0,0.0, sin(y),0.0,cos(y));
    }
     
    float Hash (float p)
    {
        p = frac(p * .1031);
        p *= p + 33.33;
        p *= p + p;
        return frac(p);
    }
     
    Vertex GenerateQuad (uint id) // generate vertex for grid of quads, from 1D array index (branchless version)
    {
        float instance = floor(id / 6.0); // index of current quad
        float divider = sqrt(_TriangleCount * 0.5); // "divider" can be precalculated on the script side, for maximum performance
        float3 center = float3(Mod(instance, divider), 0.0, floor(instance / divider)); // center of current quad
        Vertex vertex;
        float u = Mod(float(id), 2.0);
        float v = sign(Mod(126.0, Mod(float(id), 6.0) + 6.0));
        float3 localPos = float3(sign(u) - 0.5, sign(v) - 0.5, 0.0);
        localPos.z += sign(v) * sin(_Time * Hash(instance + 123.0) * 0.5); // grass wind animation
        vertex.position = mul(RotationY(radians(Hash(instance) * 180.0)), localPos) + center; // position with random rotation
        vertex.texcoord = float2(u, v);
        return vertex;
    }
     
    [numthreads(64, 1, 1)]
    void CSMain(uint3 threadID : SV_DispatchThreadID)
    {
        uint id = threadID.x;
        if ((int)id >= _TriangleCount) return;
        uint idx1 = id * 3;
        uint idx2 = id * 3 + 1;
        uint idx3 = id * 3 + 2;  
        Vertex v1 = GenerateQuad(idx1);
        Vertex v2 = GenerateQuad(idx2);
        Vertex v3 = GenerateQuad(idx3);  
        float3 p1 = v1.position;
        float3 p2 = v2.position;
        float3 p3 = v3.position;
        float2 uv1 = v1.texcoord;
        float2 uv2 = v2.texcoord;
        float2 uv3 = v3.texcoord;
        float3 normal = normalize(cross(p2 - p1, p3 - p2));  
        if (Mod(float(id), 2.0) == 1.0)
        {
            Store3(_VertexBuffer, idx1, p1);
            Store3(_VertexBuffer, idx2, p2);
            Store3(_VertexBuffer, idx3, p3);
            Store2(_TexcoordBuffer, idx1, uv1);
            Store2(_TexcoordBuffer, idx2, uv2);
            Store2(_TexcoordBuffer, idx3, uv3);      
        }
        else
        {
            Store3(_VertexBuffer, idx1, p2);
            Store3(_VertexBuffer, idx2, p1);
            Store3(_VertexBuffer, idx3, p3);
            Store2(_TexcoordBuffer, idx1, uv2);
            Store2(_TexcoordBuffer, idx2, uv1);
            Store2(_TexcoordBuffer, idx3, uv3);
            normal = -normal;      
        }
        Store3(_NormalBuffer, idx1, normal);
        Store3(_NormalBuffer, idx2, normal);
        Store3(_NormalBuffer, idx3, normal);  
    }
```

# Shader
```
    Shader "ProceduralGrass"
    {
        Properties
        {
            _MainTex ("MainTex ", 2D) = "white" {}
        }
        Subshader
        {
            Cull Off
            CGPROGRAM
            #pragma surface SurfaceShader Standard fullforwardshadows addshadow
            #pragma target 5.0
     
            sampler2D _MainTex;
     
            struct Input
            {
                float2 uv_MainTex;
            };
     
            void SurfaceShader (Input IN, inout SurfaceOutputStandard o)
            {
                float4 color = tex2D(_MainTex, IN.uv_MainTex);
                if (color.a < 0.5) discard;
                o.Albedo = color;
            }
     
            ENDCG
        }
    }
```