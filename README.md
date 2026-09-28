# xiaomi-dada-manifest

`repo` manifest for the Xiaomi 15 (`dada`, SM8750/Pakala) UEFI workspace.

## Checkout

```bash
repo init -u https://github.com/wumai2580/xiaomi-dada-manifest -b main -m default.xml
repo sync -j8
```

## Layout

```
dada/                     <- wumai2580/xiaomi-dada-uefi (workspace files + delta + tools)
uefi/mu_aloha_platforms/  <- Project-Aloha workspace at pinned commit
  MU_BASECORE/            <- microsoft/mu_basecore        @ 13e2fc5 (release/202502)
  Common/MU/              <- microsoft/mu_plus            @ 8b646ce (release/202502)
  Common/MU_TIANO/        <- microsoft/mu_tiano_plus      @ 2dd1beb (release/202502)
  Common/MU_OEM_SAMPLE/   <- microsoft/mu_oem_sample      @ e798945 (release/202502)
  Silicon/Arm/MU_TIANO/   <- microsoft/mu_silicon_arm_tiano @ e1f0860
  Binaries/               <- Project-Aloha/SurfaceDuoBinaries-fork @ ce59dcf
  Platforms/{SurfaceDuoACPI,CranePkg,OpensslPkg/...}
  Features/{DFCI,CONFIG}
```

## Apply the dada delta

```bash
cd uefi/mu_aloha_platforms
git apply ../../dada/delta/00-workspace.patch
git -C MU_BASECORE          apply ../../dada/delta/10-basecore.patch
git -C Common/MU            apply ../../dada/delta/11-plus.patch
git -C Silicon/Arm/MU_TIANO apply ../../dada/delta/12-silicon-arm.patch
cp -a ../../dada/workspace/. .
```

## Build

```bash
stuart_setup  -c Platforms/PakalaPkg/PlatformBuildNoSb.py
stuart_update -c Platforms/PakalaPkg/PlatformBuildNoSb.py
stuart_build  -c Platforms/PakalaPkg/PlatformBuildNoSb.py \
              TARGET=DEBUG TARGET_DEVICE=xiaomi-dada
make -C BootShim v2-d1
python3 ../../dada/tools/repack_d1.py   # adjust paths for your layout
```

Boot with `fastboot boot` only — no flashing.
