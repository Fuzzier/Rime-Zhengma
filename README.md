# 概述

簡化字單字鄭碼輸入法。

包含GB 18030-2022中的雙字節2、3、4區、康熙部首、CJK-A區的漢字單字。

共27814個漢字的鄭碼編碼。

使用`simplifier`時，使用RIME自帶的[朙月拼音]等方案進行編碼反查（臨時切換爲朙月拼音，查詢鄭碼編碼）時，总是會優先显示繁体字。

爲此，附帶提供[簡化拼音]方案，提供單字拼音編碼，單字依漢字字頻排序，鄭碼輸入法可以通過反查[簡化拼音]方案實現簡化字的鄭碼編碼反查。

# 配置方法汇总

RIME輸入法架構具有較強的可配置性，但還遠達不到隨心所欲的程度。
特別是使用快捷鍵切換狀態時，狀態不會被記憶，十分怪異的設計，切換回輸入法後必須重新切換，對於需要頻繁切換中/英文模式的應用程式非常不友好。

又如，在調用ImmSetOpenStatus(hWnd, 0)後，輸入法直接犯傻，無法通過按鍵恢復輸入狀態。

在不同輸入階段、用戶界面區域等上下文中對輸入法的控制能力也比較差。

例如，RIME目前只提供了簡單判斷條件下的擊鍵與操作的映射。
同樣是[回車]鍵，在不同上下文中，用户无法控制输入法是清屏还是上屏。

## 四碼頂字上屏

```yaml
  speller :
    max_code_length : 4
    auto_select     : true
```

## 空格上屏

```yaml
  speller:
    use_space : true
```

## 回车清除不上屏

```yaml
  key_binder :
    bindings :
      - when   : composing
        accept : Return
        send   : Escape
```

## 單字輸入

碼表中只含單字的編碼；或者使用`single_char_filter`。

```yaml
  filters :
    -single_char_filter
```

## 空碼不上屏、自動清除

暫時無解。

## 逐漸提示

```yaml
  translator :
    enable_completion : true
```

## 右Shift切換中英文，英文上屏

```yaml
  processor :
    - ascii_composer
    - express_editor
  editor:
    bindings:
      Shift_R : commit_raw_input  # 輸出已鍵入的英文字符
  ascii_composer:
    switch_key:
      Shift_L: inline_ascii  # 切換到英文輸入模式
      Shift_R: commit_text   # 切換到英文輸入模式
```

## 繁簡切換

```yaml
  switches:
    - name: simplification
      states: ["漢字", "汉字"]
  key_binder :
    bindings :
      - when   : always
        accept : Control+j
        toggle : simplification
```

## 字符集切換

```yaml
  switches:
    - name: extended_charset
      states: ["通用", "增廣"]
  key_binder :
    bindings :
      - when   : always
        accept : Control+m
        toggle : extended_charset
```

## 全角半角切換

```yaml
  switches:
    - name: full_shape
      states: ["半角", "全角"]
  key_binder :
    bindings :
      - when   : always
        accept : Shift+space
        toggle : full_shape
```

## 標點符號切換

```yaml
  switches:
    - name: ascii_punct
      states: ["句讀", "符號"]
  key_binder :
    bindings :
    - when   : always
      accept : Control+period
      toggle : ascii_punct
```

## 編碼反查（臨時使用其它輸入方案，同時提示當前方案中的編碼）

```yaml
  processors :
    - recognizer
  segmentors :
    - matcher
  translators :
    - reverse_lookup_translator

  recognizer :
    - reverse_lookup : "`[a-z]*'?$"  # 以`開啓編碼反查，以'結束編碼反查

  reverse_lookup :
    prefix : "`"   # 配合recognizer，以`开启编码反查
    suffix : "'"   # 配合recognizer，以'结束编码反查
    dictionary : simp_pinyin        # 反查翻譯器將調取此字典文件，RIME必須部署了該輸入方案
    preedit_format :                # 輸入其它方案的編碼时，將編碼轉換爲更易讀的形式
      - "xform/([nl])v/$1ü/"        # 例如，將全拼編碼"nv"顯示爲"nü"
      - "xform/([nl])ue/$1üe/"
      - "xform/([jqxy])v/$1u/"
    enable_completion : true        # 允許其它輸入方案的編碼結果上屏？
    # comment_format :
    #  - "xlit/hspnz/一丨丿丶乙/"   # 顯示當前輸入方案的編碼时，将編碼轉換爲更易讀的形式
    #                               # 例如，將筆畫編碼"hspnz"顯示爲"一丨丿丶乙"（橫堅撇捺折）
    tips: "〔全拼〕"
```

## 編碼混打（依賴於編碼反查）

```yaml
  segmentors :
    - abc_segmentor
  abc_segmentor :
    extra_tags:         # 給編碼打上額外的標籤，實現編碼混打
      - reverse_lookup  # 與反查碼混打
```

