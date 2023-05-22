# SGX SSA 

可以看一看 sgx-step、openenclave 关于这部分的代码。

Intel 手册也有这部分的描述 "STATE SAVE AREA (SSA) FRAME"


## State Save Area (SSA) Frame 

When an AEX occurs while running in an enclave, the context (namely cpu register values) is saved in the **thread's current SSA frame**, which is pointed to by **TCS.CSSA** (CSSA). 

An SSA frame must be **page aligned** (namely 4096 KB for a SSA frame),  it cotains the following 4 main regions. 

- `XSAVE` region: this region contains extened feature register state 
- Pad region: a gap between XSAVE and MISC region 
- MISC region: may contain zero or more components of extended information that would be saved when an AEX occurs. 
  - If the MISC region is absent, the region between the GPRSGX and XSAVE regions is the pad region.
  - If the MISC region is present, the region between the MISC and XSAVE regions is the pad region
- `GPRSGX` region: general purpose registers (RAX … R15), the RIP, the outside RSP and RBP, RFLAGS and the AEX information

![](2023-05-22-23-18-22.png)


## Sgx-Step  

```c
uint64_t edbgrd_ssa_gprsgx(int gprsgx_field_offset)
{
    uint64_t ret;
    void *ssa_field_addr = get_enclave_ssa_gprsgx_adrs() + gprsgx_field_offset;
    edbgrd(ssa_field_addr, &ret, 8);

    return ret;
}

void* get_enclave_ssa_gprsgx_adrs(void)
{
    uint64_t ossa = 0x0;
    uint32_t cssa = 0x0;
    void *tcs_addr = sgx_get_tcs();
    edbgrd(tcs_addr + SGX_TCS_OSSA_OFFSET, &ossa, sizeof(ossa));
    edbgrd(tcs_addr + SGX_TCS_CSSA_OFFSET, &cssa, sizeof(cssa));

    return get_enclave_base() + ossa + (cssa * SGX_SSAFRAMESIZE) - SGX_GPRSGX_SIZE;
}
```

其中 `edbgrd_ssa_gprsgx` 是读取 SSA gprsgx 区域的内容。

可以看到 `get_enclave_ssa_gprsgx_adrs` 返回 SSA 的 GPRSGX 地址。




