# Rime 个人配置

个人自用的 Rime 词库与方案配置。Linux（fcitx5-rime）与 Windows（小狼毫 Weasel）共用同一份文件。

## 目录结构

| 路径 | 作用 |
|---|---|
| `lazyabel.dict.yaml` | **主词表**。只放自己的词条，公共词库全部 import 进来 |
| `luna_pinyin*.custom.yaml` | 朙月拼音系列方案的补丁：把 `translator/dictionary` 指向 `lazyabel`（`luna_pinyin` 与 `luna_pinyin_simp` 另有 emoji 开关 `show_es`） |
| `default.custom.yaml` | 方案列表（6 个）、菜单每页 7 条、切换热键 |
| `weasel.custom.yaml` | 小狼毫皮肤设置，**仅 Windows 读取**，Linux 下无副作用 |
| `dicts/` | 自制与第三方词库，见下表 |
| `luna_pinyin.{extended,hanyu,poetry,cn_en}.dict.yaml` | 瑾昀的朙月拼音扩充词库（上游原文件，未改动） |
| `opencc/emoji_symbols.{json,txt}` | `es_conversion` 滤镜用的中文 → emoji 对照表 |

`dicts/` 内容：

| 文件 | 来源 | 条数 |
|---|---|---|
| `IT.dict.yaml` | THUOCL 科技词库 + 手工补充的编程词汇 | 12.8k |
| `animal.dict.yaml` | THUOCL 动物词库（前 3565 条有权重）+ 一批零权重物种名 | 17.3k |
| `place_and_people.dict.yaml` | THUOCL 地名 + 历史名人 | 58.3k |
| `symbols.dict.yaml` | 希腊字母 / 数学符号，`sort: original` | 154 |
| `chinese_classic.dict.yaml` | 二十八宿、四时田猎、《秦妇吟》节选 | 75 |

## 词表是怎么串起来的

```
输入编码
  └─ 方案 schema（luna_pinyin / luna_pinyin_simp / luna_pinyin_tw / luna_pinyin_fluency，四者共用同一张词表）
       └─ translator/dictionary = lazyabel      ← 由各 *.custom.yaml 指定
            ├─ luna_pinyin.extended             （本地文件，已含下列四张）
            │    ├─ luna_pinyin                 （系统自带 + essay.txt 八股文）
            │    ├─ luna_pinyin.hanyu
            │    ├─ luna_pinyin.poetry
            │    └─ luna_pinyin.cn_en
            └─ dicts/{animal, IT, place_and_people, symbols, chinese_classic}
```

`cangjie5`（仓颉，字形码）和 `terra_pinyin`（带声调的地球拼音）编码体系不同，**不**共用 `lazyabel`。

## 加词

在 `lazyabel.dict.yaml` 的 `...` 之后加一行，三列用 TAB 分隔：

```
文字	py lin	20000
```

- 拼音用朙月拼音音节，空格分隔；`ü` 写作 `v`（`女 nv`、`綠 lv`）。
- 加词后必须重新部署才生效。
- **拼音必须是合法音节，否则这个词永远打不出来且不会报错。** 例如 `zhaung` 不是音节（应为 `zhuang`），那种词在词库里等于不存在。
- 同一个词可以有多个编码（多音字、简拼），各写一行即可；同一编码下多个词按权重排序。

## 权重

权重是**绝对优先级**，会和所有词表（含 `essay.txt`，最高可达数百万）直接比较。所以本表按档位换算使用：

| 档位 | 实际权重 | 大致效果 |
|---|---|---|
| 1 | 20 000 | 压过 `金谷`(473)、`石佛`(408)、`鹮`(5069) 这类中低频词 |
| 2 | 40 000 | 仍低于 `黄蜂`(118861) 这类高频词 |
| 5 | 100 000 | |
| 8 | 160 000 | 可压过高频词 |

新档位 = 档位 × 20000。想临时插一条最优先的词，直接写 `160000` 以上即可。

## 部署

Linux（fcitx5-rime）。**注意：本机 `fcitx5-remote -r` 会返回成功但并不会真的重启进程**，必须杀进程重起：

```nu
# 1. 强迫重建（可选，改过 dict 时建议做）
rm -f build/lazyabel.{table,reverse,prism}.bin

# 2. 重启 fcitx5
pkill -x fcitx5; sleep 3
nohup setsid fcitx5 > /tmp/fcitx5.log 2>&1 &

# 3. 确认编译结果
sleep 8; ls -l build/lazyabel.table.bin
```

`lazyabel.table.bin` 正常约 **21 MB**。如果只有几 MB，说明词表没编译完（多半是部署被别的操作打断），此时打不出中文，删掉产物重来一次即可。

Windows：托盘图标右键 → 重新部署。

## 不入库的文件

`build/`（编译出的 `.bin`）、`*.userdb/`（用户学习词典）、`sync/`（自动同步快照，含 installation_id）、`installation.yaml` 与 `user.yaml`（本机运行状态）。这些每台机器各自生成，入库只会冲突。
