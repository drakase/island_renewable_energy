# 離島にある再生可能エネルギー発電設備の調査ツール

## 概要
* 経済産業省 資源エネルギー庁が提供する再生可能エネルギー発電設備のデータと、[離島住所生成ツール](https://github.com/drakase/island_address)で生成した離島住所(都道府県 + 市町村名 + 丁目)一覧を使用して、離島の発電事業者が運転する発電設備の一覧や割合等を生成します

## 入出力データ
### 入力データ
* [再生可能エネルギー発電事業計画の認定情報](https://www.fit-portal.go.jp/PublicInfo)
  * 再生可能エネルギー発電設備の住所等を得るために必要です
  * 例えば、長崎県の離島の発電事業者が運転する発電設備の割合等を算出する場合は以下のデータをダウンロードします
    * https://www.fit-portal.go.jp/servlet/servlet.FileDownload?retURL=%2Fapex%2FPublicInfo&file=00P0K00002D8YRvUAN
* 離島住所一覧
  * [離島住所生成ツール](https://github.com/drakase/island_address)を使用して生成します
  * 発電事業者や発電設備の住所が離島内であるかを判定するために必要です
    * 都道府県単位で離島住所(都道府県 + 市町村名 + 丁目)一覧を生成します
### 出力データ
* 以下のようなCSV形式で離島の発電事業者が運転する発電設備の一覧等が保存されます(1行目はヘッダ)
  * フラグの説明
    * 島内事業者フラグ=1 かつ 島内発電設備フラグ=1: 離島内の発電事業者が運転する離島内の発電設備
    * 島内事業者フラグ=0 かつ 島内発電設備フラグ=1: 離島外の発電事業者が運転する離島内の発電設備
    * 島内事業者フラグ=1 かつ 島内発電設備フラグ=0: 離島内の発電事業者が運転する離島外の発電設備
    * 島内事業者フラグ=0 かつ 島内発電設備フラグ=0: 離島外の発電事業者が運転する離島外の発電設備
  * この例では業者名や住所の一部を`***`で隠していますが、実際の処理結果では隠されません
```csv
設備ID,発電事業者名,代表者名,事業者の住所,事業者の電話番号,発電設備区分,発電出力（kW）,発電設備の所在地,島内事業者,島内発電設備
2C33******,五島****合同会社,****建設株式会社,長崎県五島市****,NULL,洋上風力,16800,長崎県五島市上大津町****,1,1
A657******,株式会社****,馬****,熊本県熊本市中央区船場町****,NULL,太陽光,250,長崎県雲仙市吾妻町栗林名****,0,0
A678******,株式会社****,平****,諫早市正久寺町****,095****,太陽光,47.6,長崎県諫早市正久寺町****,0,0
...
A732******,****株式会社,粥****,福岡市東区西戸崎****,092****,太陽光,1500,長崎県長崎市高島町****,0,1
...
A728******,****有限会社,谷****,五島市福江町****,095****,太陽光,42,長崎県長崎市西海町****,1,0
```
* また、以下のようなJSON形式で離島の発電事業者が運転する発電設備の割合等が保存されます
  * 再生可能エネルギー発電設備の割合は、事業者の住所または発電設備の所在地が登録されていない(NULL)レコードを無視して算出しています
```json
{
  "再生可能エネルギー発電設備数": 7406,
  "再生可能エネルギー発電設備の割合: 発電事業者が離島内かつ発電設備の所在地が離島内 / 発電設備の所在地が離島内": "273 / 523 = 52.199%",
  "再生可能エネルギー発電設備の割合: 発電事業者が離島外かつ発電設備の所在地が離島内 / 発電設備の所在地が離島内": "250 / 523 = 47.801%"
}
```

## 実行方法
* Python(バージョン3.10以上推奨)がインストールされた環境で以下を実行してください
  * ここでは長崎県の例を示します
```bash
git clone https://github.com/drakase/island_renewable_energy.git
cd island_renewable_energy

# 【初回のみ】実行環境を構築します
. ./setup.sh

# 適宜ディレクトリを作成して、入力データを配置します
# 長崎県の離島住所一覧: island_address/42_nagasaki.csv
# 長崎県の再生可能エネルギー発電事業計画の認定情報: renewable_energy_operator/42.長崎県_202308.xlsx

# 都道府県単位で離島の発電事業者が運転する発電設備の一覧や割合等を生成します
. ./check_operator.sh \
    island_address/42_nagasaki.csv \
    renewable_energy_operator/42.長崎県_202308.xlsx \
    island_renewable_energy_operator/42_nagasaki.csv

# この例では、island_renewable_energy_operator/42_nagasaki.csv に長崎県の離島の発電事業者が運転する発電設備の一覧(フラグ)等が保存されます
# また、island_renewable_energy_operator/42_nagasaki.json に長崎県の離島の発電事業者が運転する発電設備の割合等が保存されます
```
### 一括実行
```bash
# 適宜ディレクトリを作成して、全ての都道府県のデータを配置します
# 離島住所一覧: island_address/*.csv
# 再生可能エネルギー発電事業計画の認定情報: renewable_energy_operator/*.xlsx

# 離島振興対策実施地域データが存在する全ての都道府県の離島の発電事業者が運転する発電設備の一覧等を生成します
for pref_no in $(ls island_address | grep .csv | grep -v all.csv | sed -E "s/^(.*).csv/\1/"); do
  island_address=${pref_no}.csv
  renewable_energy_operator=`ls renewable_energy_operator | grep -E "^${pref_no}\..+\.xlsx$"`
  out_file_name=${pref_no}.csv
  if [ ! -e island_renewable_energy_operator/$out_file_name ]; then
    . ./check_operator.sh \
        island_address/$island_address \
        renewable_energy_operator/$renewable_energy_operator \
        island_renewable_energy_operator/$out_file_name
  fi
done

# 全ての都道府県の離島の発電事業者が運転する発電設備の一覧を生成します
awk 'FNR!=1||NR==1' island_renewable_energy_operator/??.csv > island_renewable_energy_operator/all.csv

# 全ての都道府県の離島の発電事業者が運転する発電設備の割合等を保存します(all.json が作成されます)
. ./check_operator.sh "NULL" "NULL" island_renewable_energy_operator/all.csv
```
### island_renewable_energy_operator/all.json
```json
{
  "再生可能エネルギー発電設備数": 217217,
  "再生可能エネルギー発電設備の割合: 発電事業者が離島内かつ発電設備の所在地が離島内 / 発電設備の所在地が離島内": "426 / 1439 = 29.604%",
  "再生可能エネルギー発電設備の割合: 発電事業者が離島外かつ発電設備の所在地が離島内 / 発電設備の所在地が離島内": "1013 / 1439 = 70.396%"
}
```
* __離島にある発電設備のうち、約3割が離島の発電事業者により運転されているようです__
