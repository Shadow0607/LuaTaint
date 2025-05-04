# LuaTaint
An automated static taint analysis tool for the Lua web framework.

主要流程，下載bin 檔，我是用kali linux的虛擬機跑的，測試用我手邊一台沒再用的tplink deco m4

FW_FILE="M4_4.0_en_1.1.2_Build_20231227_Rel.84005.bin"
EXTRACT_DIR="_$(basename "$FW_FILE").extracted"
TAINT_OUT_DIR="taint_results_$(basename "$FW_FILE" .bin)"
echo "📦 binwalk 開始解開韌體..."
sudo binwalk --run-as=root -e "$FW_FILE"
mkdir -p "$TAINT_OUT_DIR"
echo "🔍 開始分析 Lua 腳本..."

find "$EXTRACT_DIR" -type f \( -iname "*.lua" -o -iname "*.luac" -o -perm -u+x \) | while read file; do
  if grep -q "os.execute" "$file"; then
    fname=$(basename "$file")
    echo "🧪 分析中: $fname"

    # 如果是 luac，反編譯
    if [[ "$file" == *.luac ]]; then
      luadec "$file" > "/tmp/$fname.lua"
      ANALYZE_FILE="/tmp/$fname.lua"
    else
      ANALYZE_FILE="$file"
    fi

    # 執行 LuaTaint（你需先修正好 LuaTaint 的錯誤）
    python3 ~/LuaTaint/__main__.py "$ANALYZE_FILE" > "$TAINT_OUT_DIR/$fname.log" 2>&1

    echo "✅ 分析完成 → $TAINT_OUT_DIR/$fname.log"
  fi
done
echo "🎉 所有分析完成，請查看 → $TAINT_OUT_DIR/"
