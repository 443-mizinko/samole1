# samole1
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>相乗りタクシー マッチング（検証用）</title>

  <!-- Googleサイト iframe 埋め込み前提 -->
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    body {
      margin: 0;
      padding: 16px;
      font-family: Arial, sans-serif;
      background: #ffffff;
      box-sizing: border-box;
    }

    h1 {
      font-size: 20px;
      margin-bottom: 12px;
    }

    .box {
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 12px;
      margin-bottom: 16px;
    }

    label {
      display: block;
      margin-bottom: 10px;
      font-size: 14px;
    }

    input {
      width: 100%;
      padding: 6px;
      font-size: 14px;
      margin-top: 4px;
    }

    button {
      width: 100%;
      padding: 10px;
      font-size: 15px;
      background: #1a73e8;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }

    button:hover {
      background: #1558b0;
    }

    .result {
      font-size: 14px;
      line-height: 1.6;
    }
  </style>
</head>
<body>

<h1>相乗りタクシー マッチング（検証デモ）</h1>

<div class="box">
  <label>
    あなたのグループ人数
    <input type="number" id="myGroup" value="1" min="1">
  </label>

  <label>
    目的地までの距離（m）
    <input type="number" id="myDistance" value="300">
  </label>

  <button onclick="match()">相乗りを探す</button>
</div>

<div class="box result" id="result">
  条件を入力して「相乗りを探す」を押してください。
</div>

<script>
  /*
    Googleサイト用 注意点
    ・外部ライブラリなし
    ・1ファイル完結
    ・iframe表示でも崩れない設計
  */

  const BASE_FARE = 1800;

  // 仮の利用者データ（検証用）
  const candidates = [
    { name: "a", group: 2, distance: 300 },
    { name: "b", group: 1, distance: 250 },
    { name: "c", group: 4, distance: 300 }
  ];

  function match() {
    const myGroup = Number(document.getElementById("myGroup").value);
    const myDistance = Number(document.getElementById("myDistance").value);
    const result = document.getElementById("result");

    // 入力検証
    if (myGroup <= 0 || myDistance <= 0) {
      result.textContent = "入力値が不正です。";
      return;
    }

    // 距離差500m以内のみ対象
    const filtered = candidates.filter(c =>
      Math.abs(c.distance - myDistance) <= 500
    );

    if (filtered.length === 0) {
      result.textContent = "条件に合う相乗り相手がいません。";
      return;
    }

    // 評価関数：支払額 + 距離ペナルティ
    const scored = filtered.map(c => {
      const diff = Math.abs(c.distance - myDistance);
      const myFare = BASE_FARE / (myGroup + c.group);
      const score = myFare + diff * 0.5;
      return { ...c, diff, myFare, score };
    }).sort((a, b) => a.score - b.score);

    const best = scored[0];

    result.innerHTML = `
      <strong>マッチ結果</strong><br><br>
      相手：${best.name}<br>
      相手グループ人数：${best.group}人<br>
      距離差：${best.diff}m<br>
      あなたの支払額：${Math.round(best.myFare)}円<br><br>
      <em>※料金と距離を総合評価して選定</em>
    `;
  }
</script>

</body>
</html>
