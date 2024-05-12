# Chunked Formats

## General Block format

```cpp
struct Block {
    unsigned char block_id[4],
    uint32_t size,
    unsigned char data[size],
}

template<typename T>
struct Block {
    unsigned char block_id[4],
    uint32_t size,
    T data,
}
```

## Block IDs

| File ID | Chunk IDs                                                                      |
| ------- | ------------------------------------------------------------------------------ |
| AMC     | AMC, CMSH, QUAD                                                                |
| CM3     | ANI, CM3, EVA, NAE, NAM, SCN                                                   |
| DUM     | DUM, INI                                                                       |
| EMI     | EMI, LFVF, MAP, MAT, TRI                                                       |
| SM3     | ANI, CAM, INI, LFVF, LUZ, MAP, MAT, MD3D, NAE, NAM, PORT, SCN, SM3, SPR3, SUEL |

Read types:

- `i`: 4-byte unsigned integer
- `s`: 4-byte signed integer
- `p`: length prefixed string
- `f`: 4-byte float
- `3f`: array of 3 4-byte floats
- `3i`: array of 3 4-byte unsigned integers

| Chunk ID | Description             | Reads                    |
| -------- | ----------------------- | ------------------------ |
| AMC      | Collision Data          |                          |
| ANI      | Animation data?         |                          |
| CAM      | Camera info?            |                          |
| CMSH     | Collision Mesh Data     |                          |
| EVA      | Vertex animation data   |                          |
| DUM      | Dummy (map object) data |                          |
| INI      | INI-Configuration data  |                          |
| LFVF     | FVF Vertex Data         |                          |
| LUZ      | Lighting information    |                          |
| MAP      | Lightmap                |                          |
| MAT      | Material information    |                          |
| NAE      | Animation Data?         |                          |
| NAM      | Animation Data?         |                          |
| PORT     | Map portals             | `i==1, i, i, 4, 4`       |
| QUAD     | Mesh data?              |                          |
| SCN      | Scene tree data         |                          |
| SUEL     | Ground Plane?           | 0x18, 0xc, 4, 4, 4, 0x18 |
| TRI      | Triangle Mesh           |                          |
| MD3D     | 3D Model definition     |                          |
| EMI      | Level geometry          |                          |

## Format of Specific chunks

### INI

Configuration Data

```cpp
struct INI {
    uint32_t num_sections;
    struct {
        uint32_t num_lines;
        struct {
            uint32_t num_chars;
            char line[num_chars]
        } lines[num_lines];
    } sections[num_sections];
};
```

### LFVF

DirectX Flexible Vertex Format Data

```cpp
struct Vertex { // fields according to flags
    float position[3]; // D3DFVF_XYZ | D3DFVF_XYZRHW | D3DFVF_XYZB*
    // float rhw; // unused even with D3DFVF_XYZRHW
    float weights[3]; // D3DFVF_XYZB*
    float normal[3]; // D3DFVF_NORMAL
    float point_size; // D3DFVF_PSIZE
    uint32_t diffuse; // D3DFVF_DIFFUSE, RGBA
    uint32_t specular; //D3DFVF_SPECULAR, RGBA
    float tex_coords[D3DFVF_TEXTUREFORMAT][D3DFVF_TEX]; // D3DFVF_TEX* and D3DFVF_TEXTUREFORMAT*
};

struct LFVF {
    uint32_t version;
    uint32_t num_entries;
    struct {
        uint32_t FVF; // FVF vertex data configuration
        uint32_t vert_size; //?,
        uint32_t num_verts;
        Vertex vertices[num_vers];
    } entry[num_entries];
};
```

### DUM

Map object data

```cpp
struct DUM {
    uint32_t unk_1;
    uint32_t num_dummies;
    uint32_t unk_2;
    struct {
        uint32_t name_length;
        char name[name_length];
        float position[3];
        float rotation[3];
        uint32_t has_ini;
        if (has_ini) {
            Block<INI> ini;
        };
        uint32_t unk_1; // has_next?
    } sections[num_sections];
};
```

### MAP

```cpp
struct MAP {
    uint32_t version;
    uint32_t tex_name_len;
    char tex_name[tex_name_len];
    // TODO: rest
}
```

### SCN

- Tree structure

```text
Escena: Models/Chars/Dtritus/Dtritus.M3D
  _raiz_escena                                       -1   0  -1  c:(null)    f:00000001  a:0000
    DC_Root                                           0   1  -1  c:(null)    f:00010090  a:0000
      DC_Camera                                       1  93  -1  c:(null)    f:00420090  a:0000
      DC_Floor                                        2  94  -1  c:(null)    f:00420090  a:0000
      Bip Detritus MASTER                             4   2  -1  c:(null)    f:00200090  a:0000
        Bip Detritus                                  5   3  -1  c:(null)    f:00200190  a:0000
          Bip Detritus Pelvis                         7   4  -1  c:(null)    f:00300190  a:0000
            Bip Detritus Spine                        8   5  -1  c:(null)    f:00300190  a:0000
              Bip Detritus Spine1                     9   6  -1  c:(null)    f:00300190  a:0000
                Bip Detritus Spine2                  10   7  -1  c:(null)    f:00300190  a:0000
                  Bip Detritus Neck                  11   8  -1  c:(null)    f:00300190  a:0000
                    Bip Detritus Head                12   9  -1  c:MallaD3D  f:00300110  a:0000  [skin]
                      Bip Detritus Ponytail1         14  10  -1  c:(null)    f:00300190  a:0000
                        Bip Detritus Ponytail11      15  11  -1  c:(null)    f:00380190  a:0000
                          Bip Detritus Ponytail12    16  12  -1  c:(null)    f:00380190  a:0000
                      Bip Detritus Ponytail2         18  13  -1  c:(null)    f:00300190  a:0000
                        Bip Detritus Ponytail21      19  14  -1  c:(null)    f:00300190  a:0000
                      BipBone Detritus MechonDer_01    23  15  -1  c:(null)    f:00300190  a:0000
                        BipBone Detritus MechonDer_02      24  16  -1  c:(null)    f:00380190  a:0000
                          BipBone Detritus MechonDer_03        25  17  -1  c:(null)    f:00380190  a:0000
                      BipBone Detritus MechonIzq_01    27  18  -1  c:(null)    f:00300190  a:0000
                        BipBone Detritus MechonIzq_02      28  19  -1  c:(null)    f:00380190  a:0000
                          BipBone Detritus MechonIzq_03        29  20  -1  c:(null)    f:00380190  a:0000
                      R_Detritus_Cabeza-Cara&Pelo    37  21  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                        R_Detritus_Cabeza-Mechones   38  22  -1  c:MallaD3D  f:00000010  a:FF7F  [skin]
                        R_Detritus_Cabeza-Ceja_Izq   39  23  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-Ceja_Der   40  24  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-Ojo_Izq    41  25  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-Ojo_Der    42  26  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-OjoParpado_Der         43  27  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-OjoParpado_Izq         44  28  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Cabeza-PeloMechones       45  29  -1  c:MallaD3D  f:00000070  a:FF7F  [skin]
                    Bip Detritus L Clavicle          54  30  -1  c:(null)    f:00200190  a:0000
                      Bip Detritus L UpperArm        55  31  -1  c:(null)    f:00300190  a:0000
                        Bip Detritus L Forearm       56  32  -1  c:(null)    f:00300190  a:0000
                          Bip Detritus L Hand        57  33  -1  c:MallaD3D  f:00300110  a:0000  [skin]
                            Bip Detritus L Finger0   58  34  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus L Finger01      59  35  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus L Finger1   61  36  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus L Finger11      62  37  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus L Finger2   64  38  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus L Finger21      65  39  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus L Finger3   67  40  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus L Finger31      68  41  -1  c:(null)    f:00300190  a:0000
                          R_Detritus_Antebrazo_Izq   71  42  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Codo_Izq      72  43  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Antebrazo-Doblez_Izq            73  44  -1  c:MallaD3D  f:00000010  a:FF7F  [skin]
                            R_Detritus_Brazo-CilindroB-Eje_Izq               75  45  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Brazo-CilindroB_Izq           76  46  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Brazo_Izq         77  47  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                          R_Detritus_Brazo-CilindroA_Izq         79  48  -1  c:MallaD3D  f:00000010  a:0000
                    Bip Detritus R Clavicle          80  49  -1  c:(null)    f:00200190  a:0000
                      Bip Detritus R UpperArm        81  50  -1  c:(null)    f:00300190  a:0000
                        Bip Detritus R Forearm       82  51  -1  c:(null)    f:00300190  a:0000
                          Bip Detritus R Hand        83  52  -1  c:MallaD3D  f:00300110  a:0000  [skin]
                            Bip Detritus R Finger0   84  53  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus R Finger01      85  54  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus R Finger1   87  55  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus R Finger11      88  56  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus R Finger2   90  57  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus R Finger21      91  58  -1  c:(null)    f:00300190  a:0000
                            Bip Detritus R Finger3   93  59  -1  c:(null)    f:00300190  a:0000
                              Bip Detritus R Finger31      94  60  -1  c:(null)    f:00300190  a:0000
                          R_Detritus_Antebrazo_Der   97  61  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Codo_Der      98  62  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Antebrazo-Doblez_Der            99  63  -1  c:MallaD3D  f:00000010  a:FF7F  [skin]
                            R_Detritus_Brazo-CilindroB-Eje_Der              100  64  -1  c:MallaD3D  f:00000010  a:0000
                            R_Detritus_Brazo-CilindroB_Der          102  65  -1  c:MallaD3D  f:00000010  a:0000
                        R_Detritus_Brazo_Der        103  66  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                          R_Detritus_Brazo-CilindroA_Der        105  67  -1  c:MallaD3D  f:00000010  a:0000
                R_Detritus_Tronco                   106  68  -1  c:MallaD3D  f:00000010  a:FF7F  [skin]
              Bip Detritus L Thigh                  107  69  -1  c:(null)    f:00300190  a:0000
                Bip Detritus L Calf                 108  70  -1  c:(null)    f:00300190  a:0000
                  Bip Detritus L Foot               109  71  -1  c:(null)    f:00300190  a:0000
                    Bip Detritus L Toe0             110  72  -1  c:(null)    f:00300190  a:0000
                  R_Detritus_Espinilla_Izq          112  73  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                    R_Detritus_Rodilla_Izq          113  74  -1  c:MallaD3D  f:00000010  a:0000
                      R_Detritus_Pierna-CilindroB_Izq     114  75  -1  c:MallaD3D  f:00000010  a:0000
                    R_Detritus_Espinilla-CilindroA_Izq      116  76  -1  c:MallaD3D  f:00000010  a:0000
                    R_Detritus_Talon_Izq            118  77  -1  c:MallaD3D  f:00000010  a:0000
                      R_Detritus_Espinilla-CilindroB_Izq        119  78  -1  c:MallaD3D  f:00000010  a:0000
                R_Detritus_Pierna_Izq               121  79  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                  R_Detritus_Pierna-CilindroA_Izq   122  80  -1  c:MallaD3D  f:00000010  a:0000
              Bip Detritus R Thigh                  124  81  -1  c:(null)    f:00300190  a:0000
                Bip Detritus R Calf                 125  82  -1  c:(null)    f:00300190  a:0000
                  Bip Detritus R Foot               126  83  -1  c:(null)    f:00300190  a:0000
                    Bip Detritus R Toe0             127  84  -1  c:(null)    f:00300190  a:0000
                    R_Detritus_Talon_Der            129  85  -1  c:MallaD3D  f:00000010  a:0000
                      R_Detritus_Espinilla-CilindroB_Der        130  86  -1  c:MallaD3D  f:00000010  a:0000
                  R_Detritus_Espinilla_Der          132  87  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                    R_Detritus_Rodilla_Der          133  88  -1  c:MallaD3D  f:00000010  a:0000
                      R_Detritus_Pierna-CilindroB_Der     135  89  -1  c:MallaD3D  f:00000010  a:0000
                    R_Detritus_Espinilla-CilindroA_Der      136  90  -1  c:MallaD3D  f:00000010  a:0000
                R_Detritus_Pierna_Der               138  91  -1  c:MallaD3D  f:00000010  a:0000  [skin]
                  R_Detritus_Pierna-CilindroA_Der   139  92  -1  c:MallaD3D  f:00000010  a:0000
```
