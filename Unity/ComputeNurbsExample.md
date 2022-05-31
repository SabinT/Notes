# Credits

Code by Przemyslaw_Zaworski from https://forum.unity.com/threads/feedback-wanted-mesh-compute-shader-access.1096531/


# C# Code
```csharp
    using Unity.Collections;
    using UnityEngine;
    using UnityEngine.Rendering;
    using System.Collections.Generic;
     
    [RequireComponent(typeof(MeshFilter))]
    [RequireComponent(typeof(MeshRenderer))]
    public class NurbsSurface : MonoBehaviour
    {
        public ComputeShader NurbsSurfaceCS;
        [Range(1, 1024)] public int TessellationFactor = 64;
        [System.Serializable] public struct ControlPoint {public Transform Transform; public float Weight;};
        [System.Serializable] public struct Vertex {public Vector3 Position; public Vector3 Normal; public Vector2 Texcoord;};
        public ControlPoint[] ControlPoints;
     
        private List<Vector4> _ControlPoints = new List<Vector4>();
        private GraphicsBuffer _GraphicsBuffer;
        private Mesh _Mesh;
        private bool _Recalculate = false;
        private int _VertexCount = 0;
     
        void LoadDefaultSettings()
        {
            Vector3[] vectors = new Vector3[]
            {
                new Vector3(00.0f, 00.0f, 00.0f), new Vector3(10.0f, 00.0f, 00.0f), new Vector3(20.0f, 00.0f, 00.0f), new Vector3(30.0f, 00.0f, 00.0f),
                new Vector3(00.0f, 00.0f, 10.0f), new Vector3(10.0f, 10.0f, 10.0f), new Vector3(20.0f, 10.0f, 10.0f), new Vector3(30.0f, 00.0f, 10.0f),
                new Vector3(00.0f, 00.0f, 20.0f), new Vector3(10.0f, 10.0f, 20.0f), new Vector3(20.0f, 10.0f, 20.0f), new Vector3(30.0f, 00.0f, 20.0f),
                new Vector3(00.0f, 00.0f, 30.0f), new Vector3(10.0f, 00.0f, 30.0f), new Vector3(20.0f, 00.0f, 30.0f), new Vector3(30.0f, 00.0f, 30.0f)
            };
            ControlPoints = new ControlPoint[vectors.Length];
            for (int i = 0; i < vectors.Length; i++)
            {
                GameObject element = new GameObject();
                element.name = "ControlPoint" + (i + 1).ToString();
                element.transform.parent = this.transform;
                element.transform.position = vectors[i];
                ControlPoints[i] = new ControlPoint() {Transform = element.transform, Weight = 1.0f};
            }
        }
     
        /* Example for 16 control points (grid 4x4) - fill grid 4x4 into larger collection (max 64)
        HLSL arrays need to have constant (predefined) size, so for another number of control points
        you need to modify a code a bit...
            ########
            ########
            ########
            ########
            ****####
            ****####
            ****####
            ****####
        */
        void LoadDefaultCollection()
        {
            _ControlPoints.Clear();
            _ControlPoints.TrimExcess();
            int index = 0;
            for (int i = 0; i < 64; i++) // max 64 elements, because _ControlPoints[8][8] from compute shader
            {
                if (index > ControlPoints.Length - 1) continue;
                if (i % 8 > 3 || i > 31)
                {
                    _ControlPoints.Add(Vector4.zero);
                }
                else
                {
                    Vector3 p = ControlPoints[index].Transform.position;
                    _ControlPoints.Add(new Vector4(p.x, p.y, p.z, ControlPoints[index].Weight));
                    index++;
                }
            }
        }
     
        void ExportMesh()
        {
            #if UNITY_EDITOR
                Vertex[] points = new Vertex[_Mesh.vertices.Length];
                _GraphicsBuffer.GetData(points);
                Mesh mesh = new Mesh();
                mesh.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32;
                List<Vector3> vertices = new List<Vector3>();
                List<int> triangles = new List<int>();
                List<Vector3> normals = new List<Vector3>();
                List<Vector2> uvs = new List<Vector2>();
                for (int i = 0; i < points.Length; i++)
                {
                    vertices.Add(points[i].Position);
                    triangles.Add(i);
                    normals.Add(points[i].Normal);
                    uvs.Add(points[i].Texcoord);
                }
                mesh.vertices = vertices.ToArray();
                mesh.triangles = triangles.ToArray();
                mesh.normals = normals.ToArray();
                mesh.uv = uvs.ToArray();
                string fileName = System.Guid.NewGuid().ToString("N");
                UnityEditor.AssetDatabase.CreateAsset(mesh, "Assets/" + fileName + ".asset");
                GameObject target = new GameObject();
                target.name = fileName;
                target.AddComponent<MeshFilter>().sharedMesh = mesh;
                MeshRenderer renderer = target.AddComponent<MeshRenderer>();
                renderer.sharedMaterial = UnityEditor.AssetDatabase.GetBuiltinExtraResource<Material>("Default-Material.mat");
                UnityEditor.PrefabUtility.SaveAsPrefabAsset(target, "Assets/" + fileName + ".prefab");
            #endif
        }
     
        void Start()
        {
            #if UNITY_EDITOR
                Material material = this.GetComponent<Renderer>().sharedMaterial;
                if (material == null)
                {
                    material = UnityEditor.AssetDatabase.GetBuiltinExtraResource<Material>("Default-Material.mat");
                    this.GetComponent<Renderer>().sharedMaterial = material;
                }
            #endif
        }
     
        void Update()
        {
            _VertexCount = TessellationFactor * TessellationFactor * 6;
            if (_Mesh == null || _Recalculate)
            {
                Release();
                _Recalculate = false;
                _Mesh = new Mesh();
                _Mesh.name = "NURBS Surface";
                _Mesh.vertexBufferTarget |= GraphicsBuffer.Target.Raw;
                _Mesh.indexBufferTarget |= GraphicsBuffer.Target.Raw;
                VertexAttributeDescriptor[] attributes = new []
                {
                    new VertexAttributeDescriptor(VertexAttribute.Position,  VertexAttributeFormat.Float32, 3, stream:0),
                    new VertexAttributeDescriptor(VertexAttribute.Normal,    VertexAttributeFormat.Float32, 3, stream:0),
                    new VertexAttributeDescriptor(VertexAttribute.TexCoord0, VertexAttributeFormat.Float32, 2, stream:0),
                };
                _Mesh.SetVertexBufferParams(_VertexCount, attributes);
                _Mesh.SetIndexBufferParams(_VertexCount, IndexFormat.UInt32);
                NativeArray<int> indexBuffer = new NativeArray<int>(_VertexCount, Allocator.Temp);
                for (int i = 0; i < _VertexCount; ++i) indexBuffer[i] = i;
                _Mesh.SetIndexBufferData(indexBuffer, 0, 0, indexBuffer.Length, MeshUpdateFlags.DontRecalculateBounds | MeshUpdateFlags.DontValidateIndices);
                indexBuffer.Dispose();
                SubMeshDescriptor subMeshDescriptor = new SubMeshDescriptor(0, _VertexCount, MeshTopology.Triangles);
                subMeshDescriptor.bounds = new Bounds(Vector3.zero, new Vector3(1e5f, 1e5f, 1e5f));
                _Mesh.SetSubMesh(0, subMeshDescriptor);
                _Mesh.bounds = subMeshDescriptor.bounds;
                GetComponent<MeshFilter>().sharedMesh = _Mesh;
                _GraphicsBuffer = _Mesh.GetVertexBuffer(0);
            }
            LoadDefaultCollection();
            NurbsSurfaceCS.SetInt("_VertexCount", _VertexCount);
            NurbsSurfaceCS.SetInt("_TessellationFactor", TessellationFactor);
            NurbsSurfaceCS.SetBuffer(0, "_GraphicsBuffer", _GraphicsBuffer);
            NurbsSurfaceCS.SetVectorArray("_ControlPoints", _ControlPoints.ToArray());
            NurbsSurfaceCS.GetKernelThreadGroupSizes(0, out uint x, out uint y, out uint z);
            int threadGroupsX = Mathf.Min((_VertexCount + (int)x - 1) / (int)x, 65535);
            int threadGroupsY = (int)y;
            int threadGroupsZ = (int)z;
            NurbsSurfaceCS.Dispatch(0, threadGroupsX, threadGroupsY, threadGroupsZ);
            if (Input.GetKeyDown(KeyCode.R)) LoadDefaultSettings();
            if (Input.GetKeyDown(KeyCode.Space) && (_Mesh != null)) ExportMesh();
        }
     
        void Release()
        {
            if (_Mesh != null) Destroy(_Mesh);
            if (_GraphicsBuffer != null) _GraphicsBuffer.Release();
        }
     
        void OnDestroy()
        {
            Release();
        }
     
        void OnValidate()
        {
            _Recalculate = true;
        }
    }
```

# Compute Shader

```
    #pragma kernel CSMain
     
    float4 _ControlPoints[8][8];
    RWByteAddressBuffer _GraphicsBuffer;
    uint _TessellationFactor, _VertexCount;
     
    static float KnotVectors[8][2] = // knotsDim = (8, 8)
    {
        {0.0, 0.0},
        {0.0, 0.0},
        {0.0, 0.0},
        {0.0, 0.0},
        {1.0, 1.0},
        {1.0, 1.0},
        {1.0, 1.0},
        {1.0, 1.0}
    };
     
    // L. Piegl, W. Tiller, "The NURBS Book", Springer Verlag, 1997
    // http://nurbscalculator.in/
    float3 NurbsSurface (float4 cps[8][8], int2 cpsDim, float knots[8][2], int2 knotsDim, float2 uv)
    {
        const int2 degree = int2(3, 3);
        #define msize max(degree.x, degree.y)
        for (int x = 0; x < cpsDim[0]; x++)
        {
            for (int y = 0; y < cpsDim[1]; y++)
            {
                cps[x][y].xyz *= cps[x][y].w;
            }
        }
        int2 spans = int2(0, 0);
        float4 p = 0;
        [unroll(2)] for (int i = 0; i < 2; i++)
        {
            int n = knotsDim[i] - degree[i] - 2;
            if (uv[i] == (knots[n + 1][i])) spans[i] = n;
            int low = degree[i];
            int high = n + 1;
            int mid = (int)floor((low + high) / 2.0);
            [unroll(16)]
            while (uv[i] < knots[mid][i] || uv[i] >= knots[mid + 1][i])
            {
                if (uv[i] < knots[mid][i])
                    high = mid;
                else
                    low = mid;
                mid = (int)floor((low + high) / 2.0);
            }
            spans[i] = mid;
        }
        float N     [msize + 1][2];
        float left  [msize + 1][2];
        float right [msize + 1][2];
        N[0][0] = N[0][1] = 1.0;
        [loop] for (int h = 0; h < 2; h++)
        {
            float saved = 0.0, temp = 0.0;
            [loop] for (int j = 1; j <= degree[h]; j++)
            {
                left[j][h] = (uv[h] - knots[spans[h] + 1 - j][h]);
                right[j][h] = knots[spans[h] + j][h] - uv[h];
                saved = 0.0;
                [loop] for (int r = 0; r < j; r++)
                {
                    temp = N[r][h] / (right[r + 1][h] + left[j - r][h]);
                    N[r][h] = saved + right[r + 1][h] * temp;
                    saved = left[j - r][h] * temp;
                }
                N[j][h] = saved;
            }
        }
        for (int m = 0; m <= degree[1]; m++)
        {
            float4 t = 0;
            for (int k = 0; k <= degree[0]; k++) t += cps[spans[0] - degree[0] + k][spans[1] - degree[1] + m] * N[k][0];
            p += t * N[m][1];
        }
        return (p.w != 0) ? p.xyz / p.w : p.xyz;
    }
     
    float3 NurbsSurfaceNormal (float4 cps[8][8], int2 cpsDim, float knots[8][2], int2 knotsDim, float2 uv)
    {
        for (int x = 0; x < cpsDim[0]; x++) for (int y = 0; y < cpsDim[1]; y++) cps[x][y].xyz *= cps[x][y].w;
        const int order = 1; // order of the derivative
        const int2 degree = int2(3, 3); // surface degrees
        #define msize max(degree.x, degree.y)
        int2 spans = int2(0, 0);
        [unroll(2)] for (int i = 0; i < 2; i++) // find spans
        {
            int n = knotsDim[i] - degree[i] - 2;
            if (uv[i] == (knots[n + 1][i])) spans[i] = n;
            int low = degree[i];
            int high = n + 1;
            int mid = (int)floor((low + high) / 2.0);
            [unroll(16)] while (uv[i] < knots[mid][i] || uv[i] >= knots[mid + 1][i])
            {
                if (uv[i] < knots[mid][i])
                    high = mid;
                else
                    low = mid;
                mid = (int)floor((low + high) / 2.0);
            }
            spans[i] = mid;
        }
        float basis[order + 1][msize + 1][2];
        float left[msize + 1][2];
        float right[msize + 1][2];
        float ndu[msize + 1][msize + 1][2];
        ndu[0][0][0] = ndu[0][0][1] = 1.0;
        [unroll(2)] for (int q = 0; q < 2; q++) // derivatives of the basis functions
        {
            [loop] for (int j = 1; j <= degree[q]; j++)
            {
                left[j][q] = uv[q] - knots[spans[q] + 1 - j][q];
                right[j][q] = knots[spans[q] + j][q] - uv[q];
                float saved = 0.0;
                [loop] for (int r = 0; r < j; r++)
                {
                    ndu[j][r][q] = right[r + 1][q] + left[j - r][q];
                    float temp = ndu[r][j - 1][q] / ndu[j][r][q];
                    ndu[r][j][q] = saved + right[r + 1][q] * temp;
                    saved = left[j - r][q] * temp;
                }
                ndu[j][j][q] = saved;
            }
            [loop] for (int m = 0; m <= degree[q]; m++) basis[0][m][q] = ndu[m][degree[q]][q];
            float a[2][msize + 1][2];
            for (int r = 0; r <= degree[q]; r++)
            {
                int s1 = 0;
                int s2 = 1;
                a[0][0][q] = 1.0;
                [unroll(order)] for (int k = 1; k <= order; k++)
                {
                    float d = 0.0;
                    int rk = r - k;
                    int pk = degree[q] - k;
                    int j1 = 0;
                    int j2 = 0;
                    if (r >= k)
                    {
                        a[s2][0][q] = a[s1][0][q] / ndu[pk + 1][rk][q];
                        d = a[s2][0][q] * ndu[rk][pk][q];
                    }
                    j1 = (rk >= -1) ? 1 : -rk;
                    j2 = (r - 1 <= pk) ? k - 1 : degree[q] - r;
                    [unroll(order)] for (int j = j1; j <= j2; j++)
                    {
                        a[s2][j][q] = (a[s1][j][q] - a[s1][j - 1][q]) / ndu[pk + 1][rk + j][q];
                        d += a[s2][j][q] * ndu[rk + j][pk][q];
                    }
                    if (r <= pk)
                    {
                        a[s2][k][q] = -a[s1][k - 1][q] / ndu[pk + 1][r][q];
                        d += a[s2][k][q] * ndu[r][pk][q];
                    }
                    basis[k][r][q] = d;
                    int s3 = s1;
                    s1 = s2;
                    s2 = s3;
                }
            }
            float f = degree[q];
            [unroll(order)] for (int k = 1; k <= order; k++)
            {
                for (int h = 0; h <= degree[q]; h++) basis[k][h][q] *= f;
                f *= degree[q] - k;
            }
        }
        float4 derivatives[order + 1][order + 1] = {{0..xxxx, 0..xxxx}, {0..xxxx, 0..xxxx}};
        int du = min(order, degree[0]);
        int dv = min(order, degree[1]);
        float4 temp[degree[1] + 1];
        [unroll(4)] for (int w = 0; w <= du; w++) // derivatives of a B-spline surface
        {
            [unroll(4)] for (int s = 0; s <= degree[1]; s++)
            {
                temp[s] = (float4) 0;
                [unroll(4)] for (int r = 0; r <= degree[0]; r++)
                {
                    float4 pw = cps[spans[0] - degree[0] + r] [spans[1] - degree[1] + s];
                    temp[s] += pw * basis[w][r][0];
                }
            }
            int dd = min(order - w, dv);
            [unroll(4)] for (int l = 0; l <= dd; l++)
            {
                [loop] for (int s = 0; s <= degree[1]; s++) derivatives[w][l] += temp[s] * basis[l][s][1];
            }
        }
        float3 SKL[order + 1][order + 1];
        int BIN[4] = {1,1,1,1}; // binomial coefficients
        [unroll(4)] for (int k = 0; k < (order + 1); ++k) // derivatives of a NURBS surface
        {
            [unroll(4)] for (int l = 0; l < order - k + 1; ++l)
            {
                float3 v = derivatives[k][l].xyz;
                [unroll(4)] for (int z = 1; z < l + 1; ++z)
                {
                    if (z > l) continue;
                    [unroll(4)] for (int a = 1; a <= z; ++a) BIN[0] *= (l + 1 - a) / a;
                    v -= SKL[k][l - z] * derivatives[0][z].w * BIN[0];
                }
                [unroll(4)] for (int i = 1; i < k + 1; ++i)
                {
                    if (i > k)
                        BIN[1] = 0;
                    else
                        [unroll(4)] for (int b = 1; b <= i; ++b) BIN[1] *= (k + 1 - b) / b;
                    v -= SKL[k - i][l] * derivatives[i][0].w * BIN[1];
                    float3 tmp = (float3) 0;
                    [unroll(4)] for (int j = 1; j < l + 1; ++j)
                    {
                        if (j > l) continue;
                        [unroll(4)] for (int c = 1; c <= j; ++c) BIN[2] *= (l + 1 - c) / c;
                        tmp -= SKL[k - 1][l - j] * derivatives[i][j].w * BIN[2];
                    }
                    if (i > k) continue;
                    [unroll(4)] for (int d = 1; d <= i; ++d) BIN[3] *= (k + 1 - d) / d;
                    v -= tmp * BIN[3];
                }
                SKL[k][l] = v / derivatives[0][0].w;
            }
        }
        return normalize(cross(SKL[1][0], SKL[0][1]));
    }
     
    float Mod (float x, float y)
    {
        return x - y * floor(x/y);
    }
     
    void GenerateSurface (uint id : SV_VertexID, uint tess, float4 cps[8][8], float kv[8][2], out float3 position, out float3 normal, out float2 texcoord)
    {
        int instance = int(floor(id / 6.0));
        float x = sign(Mod(20.0, Mod(float(id), 6.0) + 2.0));
        float y = sign(Mod(18.0, Mod(float(id), 6.0) + 2.0));
        float u = (float(instance / tess) + x) / float(tess);
        float v = (Mod(float(instance), float(tess)) + y) / float(tess);
        float2 uv = float2(u,v);
        position = NurbsSurface (cps, int2(4, 4), kv, int2(8, 8), uv);
        normal = NurbsSurfaceNormal (cps, int2(4, 4), kv, int2(8, 8), uv);
        texcoord = uv;
    }
     
    [numthreads(64, 1, 1)]
    void CSMain(uint3 threadID : SV_DispatchThreadID)
    {
        uint id = threadID.x;
        if (id >= _VertexCount) return;
        float3 position = 0;
        float3 normal = 0;
        float2 texcoord = 0;
        GenerateSurface(id, _TessellationFactor, _ControlPoints, KnotVectors, position, normal, texcoord);
        _GraphicsBuffer.Store3((id * 8 + 0) << 2, asuint(position));
        _GraphicsBuffer.Store3((id * 8 + 3) << 2, asuint(normal));
        _GraphicsBuffer.Store2((id * 8 + 6) << 2, asuint(texcoord));
    }
```