These `llama-benchy` results are obtained with a `--mtp-draft-tokens 5` core:

```console
$ flare run $MODEL_PATH/glm-5.3-flash-hyperflare-gb10-tp2.safetensors \
  --rank 0 \
  --gpus 0 \
  --rendezvous $HEAD_IP:$HEAD_PORT \
  --host $SERVER_IP \
  --port $SERVER_PORT
         ·                                           ╱      ·    ⠂
                                            ·   ⣠⠞  ⢀⣠⣤⣤⣄⡀   ✧
                                              ⣰⡿⣠⣾⠟⠋⠉⠉⠛⢿⣦⡀   ⠄
       ████╗ ██╗                               ⣰⣿⣿⠟⠁  ⣠⠴⠒⠲⣄⠹⣷⡀
      ██╔══╝ ██║  ▄▄▄▄╗   ▄▄ ▄▄▄╗  ▄▄▄▄╗      ⢠⣿⡿⠁ ⢀⡾⠋      ⠹⡆⢹⣧ ·
     █████╗  ██║  ▄▄▄██╗  ███╔══╝ ██▄▄██╗    ⢠⣾⣿⠃ ⣠⠟   ✦      ⢻⠈⡿ ⠂
      ██╔═╝  ██║ ██╔═██║  ██╔╝    ██╔═══╝   ⢸⣿⡏ ⣰⠏           ⡼ ⠸   ╱
      ██║    ██║ ╚█████║  ██║     ╚█████╗   ⢸⣿⣿⣧⡾⠃       ⢀⡴⠋  ·  ⠊
      ╚═╝    ╚═╝  ╚════╝  ╚═╝      ╚════╝  ⣰⣿⣿⣿⡿⠁   ⢀⣠⠴⠚⠉    ⠄
                                          ⢀⣾⣿⡇ ⣠⡾⠋    ⠂    ✧
                  ⣀⣀⣀⣀⣠⠤⢤⠤⢤⡤⠤⣶⡒⢒⣒⠒⢒⡒⣶⣾⣿⣿⣷⡿⠋⡤⠤⣤⠤⠤⠤⣄⣀⣀⣀⣀
      ⢀⣀⣀⡤⢤⢴⢲⠒⡞⡽⡫⣉⢗⢦⢠⠒⡄⡴⡢⢀⢖⢆⣠⡴⠛⠋⣠⣾⡿⠿⠿⣷⣄⡔⡢⠈⠙⠲⣄⡲⡄⡔⣢⢠⢖⢄⠰⡺⡍⡝⢫⢳⢖⢖⠦⡤⢤⣀⣀⡀
⡠⢤⠔⡒⢊⡭⣭⢚⡒⢿⠶⠠⣄⣀⡠⠔⠋⣿⠖⡋⢠⠭⡄⢉⠊⣿⢖⣠⠞⠋⡀⢉⠈⠙⢦⣄⠈⠙⠷⣤⡀⡾⢒⠚⣦⠈⠢⡀⠥⡞⢓⠺⡷⢌⠁⣨⢥⡜⣓⠺⡷⢎⠁⡨⢥⡉⣛⢵⡶⢢⠤⢄
⣊⣸⠮⠞⠈⠪⠤⠤⠒⠋⠮⠼⠈⣊⣼⣮⡚⠈⣀⡠⠤⠤⠒⠋⠔ ⠣⢵⣇⣓⣁⠧⠵⠁⢓⣵⠈⠑⠢⣄⠵⠃⣓⠉⠓⠦⣄⣀⠥⣇⣑⠈⠂⠵⠃⠑⢥⣧⣑⡃⠳⠥⠇⢑⣡⣷⣕⠃⠱⠥⢇⣑

[    OK    ] Verify core.  signatures                                                                                                                                           0.4s
[    OK    ] Verify stored data.  0.0 GiB                                                                                                                                       0.0s
[    OK    ] Wait for tensor-parallel hosts.  1 peer ranks                                                                                                                      6.4s
[    OK    ] Distribute rank cores over CX7.  all rank cores verified                                                                                                           1.0s
[    OK    ] Prepare model.  1 rank                                                                                                                                             0.0s
[    OK    ] Initialize GPU.  GPU 0 · SM121                                                                                                                                     0.3s
[    OK    ] Allocate model memory.  86.3 GiB GPU · 0.2 GiB host                                                                                                                2.6s
[    OK    ] Load weights.  83.8 GiB                                                                                                                                           32.3s
[    OK    ] Prepare kernels.  4373 kernels                                                                                                                                     0.7s
[    OK    ] Prepare execution.  concurrency 4                                                                                                                                  0.6s
[    OK    ] Plan cache.  per GPU: 89.74 GiB loaded · 14.70 GiB cache budget                                                                                                    0.0s
[    OK    ] Allocate cache.  14.7 GiB · 4559360 KV tokens · 177 shared snapshot slots · limited by memory                                                                      0.9s
[    OK    ] Read model configuration.  nvidia/GLM-5.3-Flash-FP4-Local                                                                                                          0.0s
[    OK    ] Connect RoCE transport.  2 rails per peer: rank 1 roceP2p1s0f1,rocep1s0f1                                                                                          0.1s
[    OK    ] Join host prefix stores.  none                                                                                                                                     0.0s
[    OK    ] Load tokenizer.  154880 token vocabulary                                                                                                                           0.5s
[━━━━━━━━━━] Start server.                                                                                                                                                      0.5s[flare] frame completion wait: WFET
[    OK    ] Start server.  http://0.0.0.0:8000 · 1048576 tokens/request                                                                                                        0.5s

$ llama-benchy --base-url http://$SERVER_IP:$SERVER_PORT/v1 \
    --pp 2048 --tg 128 --depth 0 4096 16384 65536 100000 \
    --concurrency 1 2 4 --format md \
    --save-result BENCH.md \
    --book-url https://sherlock-holm.es/stories/plain-text/cano.txt
```

| model                          |                  test |      t/s (total) |        t/s (req) |     peak t/s |   peak t/s (req) |          ttfr (ms) |       est_ppt (ms) |      e2e_ttft (ms) |
|:-------------------------------|----------------------:|-----------------:|-----------------:|-------------:|-----------------:|-------------------:|-------------------:|-------------------:|
| nvidia/GLM-5.3-Flash-FP4-Local |           pp2048 (c1) | 1332.61 ± 287.15 | 1332.61 ± 287.15 |              |                  |   1611.86 ± 343.39 |   1609.98 ± 343.39 |   1611.86 ± 343.39 |
| nvidia/GLM-5.3-Flash-FP4-Local |           tg2048 (c1) |     38.65 ± 1.65 |     38.65 ± 1.65 | 39.00 ± 1.63 |     39.00 ± 1.63 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |           pp2048 (c2) |   1590.93 ± 9.23 |   845.68 ± 49.73 |              |                  |   2431.99 ± 143.01 |   2430.12 ± 143.01 |   2431.99 ± 143.01 |
| nvidia/GLM-5.3-Flash-FP4-Local |           tg2048 (c2) |     56.36 ± 5.26 |     29.85 ± 2.02 | 60.33 ± 2.87 |     30.17 ± 2.19 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |           pp2048 (c4) | 1371.62 ± 222.63 |   403.50 ± 49.91 |              |                  |   5179.18 ± 835.40 |   5177.30 ± 835.40 |   5179.18 ± 835.40 |
| nvidia/GLM-5.3-Flash-FP4-Local |           tg2048 (c4) |     67.20 ± 5.79 |     19.93 ± 1.51 | 81.33 ± 3.40 |     20.33 ± 1.49 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |   pp2048 @ d4096 (c1) |  2001.32 ± 13.26 |  2001.32 ± 13.26 |              |                  |    3071.99 ± 20.36 |    3070.11 ± 20.36 |    3071.99 ± 20.36 |
| nvidia/GLM-5.3-Flash-FP4-Local |   tg2048 @ d4096 (c1) |     40.11 ± 1.90 |     40.11 ± 1.90 | 40.67 ± 1.89 |     40.67 ± 1.89 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |   pp2048 @ d4096 (c2) |  1912.01 ± 56.20 |   980.01 ± 37.37 |              |                  |   6280.45 ± 242.35 |   6278.57 ± 242.35 |   6280.45 ± 242.35 |
| nvidia/GLM-5.3-Flash-FP4-Local |   tg2048 @ d4096 (c2) |     48.13 ± 1.88 |     31.51 ± 3.74 | 64.00 ± 3.56 |     32.00 ± 3.92 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |   pp2048 @ d4096 (c4) |   1937.75 ± 2.40 |   504.55 ± 13.20 |              |                  |  12187.43 ± 322.99 |  12185.56 ± 322.99 |  12187.43 ± 322.99 |
| nvidia/GLM-5.3-Flash-FP4-Local |   tg2048 @ d4096 (c4) |     67.16 ± 2.63 |     20.46 ± 1.24 | 83.67 ± 1.89 |     20.92 ± 1.38 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d16384 (c1) |  2124.33 ± 14.59 |  2124.33 ± 14.59 |              |                  |    8678.89 ± 59.86 |    8677.01 ± 59.86 |    8678.89 ± 59.86 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d16384 (c1) |     40.44 ± 3.54 |     40.44 ± 3.54 | 41.00 ± 3.74 |     41.00 ± 3.74 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d16384 (c2) |  2066.90 ± 68.84 |  1059.96 ± 36.89 |              |                  |  17413.23 ± 631.05 |  17411.36 ± 631.05 |  17413.23 ± 631.05 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d16384 (c2) |     50.68 ± 4.57 |     29.12 ± 1.89 | 59.33 ± 2.62 |     29.67 ± 1.70 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d16384 (c4) |   2111.50 ± 2.54 |    535.76 ± 5.12 |              |                  |  34408.55 ± 330.41 |  34406.67 ± 330.41 |  34408.55 ± 330.41 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d16384 (c4) |     70.94 ± 3.98 |     20.21 ± 0.91 | 82.33 ± 2.05 |     20.58 ± 0.86 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d65536 (c1) |  2162.83 ± 10.18 |  2162.83 ± 10.18 |              |                  |  31250.58 ± 147.05 |  31248.70 ± 147.05 |  31250.58 ± 147.05 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d65536 (c1) |     39.10 ± 1.44 |     39.10 ± 1.44 | 39.33 ± 1.25 |     39.33 ± 1.25 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d65536 (c2) |   2168.83 ± 1.21 |   1087.31 ± 2.91 |              |                  |  62159.52 ± 166.57 |  62157.64 ± 166.57 |  62159.52 ± 166.57 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d65536 (c2) |     49.45 ± 2.23 |     30.49 ± 2.25 | 62.00 ± 1.63 |     31.00 ± 2.16 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local |  pp2048 @ d65536 (c4) |  2154.24 ± 14.16 |    543.72 ± 3.92 |              |                  | 124306.76 ± 902.65 | 124304.89 ± 902.65 | 124306.76 ± 902.65 |
| nvidia/GLM-5.3-Flash-FP4-Local |  tg2048 @ d65536 (c4) |     68.72 ± 2.88 |     19.68 ± 1.20 | 81.00 ± 1.41 |     20.25 ± 1.16 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local | pp2048 @ d100000 (c1) |  2140.35 ± 11.73 |  2140.35 ± 11.73 |              |                  |  47681.40 ± 262.33 |  47679.52 ± 262.33 |  47681.40 ± 262.33 |
| nvidia/GLM-5.3-Flash-FP4-Local | tg2048 @ d100000 (c1) |     39.97 ± 2.42 |     39.97 ± 2.42 | 40.67 ± 2.36 |     40.67 ± 2.36 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local | pp2048 @ d100000 (c2) |   2149.54 ± 5.31 |   1076.31 ± 3.03 |              |                  |  94815.46 ± 266.97 |  94813.59 ± 266.97 |  94815.46 ± 266.97 |
| nvidia/GLM-5.3-Flash-FP4-Local | tg2048 @ d100000 (c2) |     56.95 ± 1.94 |     30.05 ± 1.35 | 61.00 ± 1.63 |     30.50 ± 1.26 |                    |                    |                    |
| nvidia/GLM-5.3-Flash-FP4-Local | pp2048 @ d100000 (c4) |   2150.16 ± 9.21 |    539.61 ± 1.81 |              |                  | 189118.52 ± 640.19 | 189116.65 ± 640.19 | 189118.52 ± 640.19 |
| nvidia/GLM-5.3-Flash-FP4-Local | tg2048 @ d100000 (c4) |     63.53 ± 3.44 |     21.93 ± 2.50 | 90.00 ± 1.63 |     22.50 ± 2.57 |                    |                    |                    |