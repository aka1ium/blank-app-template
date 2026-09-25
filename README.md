# 1. 日本語文字化け防止ライブラリのインストール
!pip install japanize-matplotlib

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import japanize_matplotlib

# ---------------------------------------------------------
# 2. サンプルデータの生成（6分野に集約）
# ---------------------------------------------------------
np.random.seed(42)  # 再現性を担保する乱数シード

students = ['Aさん', 'Bさん', 'Cさん']
days = [f"Day {i}" for i in range(1, 8)]

# 主要6分野を設定
topics_6 = [
    ('国語', '現代文'), ('国語', '古文'),
    ('数学', '数I'),   ('数学', '数A'),
    ('英語', '英単語'), ('英語', '長文読解')
]

data = []
for student in students:
    for day in days:
        # 各生徒が1日にランダムで2〜3分野を選択して学習
        num_topics = np.random.choice([2, 3])
        chosen_indices = np.random.choice(len(topics_6), size=num_topics, replace=False)
        
        for idx in chosen_indices:
            subj, topic = topics_6[idx]
            target = np.random.choice([30, 45, 60, 90, 120])  # 目標時間
            factor = np.random.uniform(0.5, 1.3)              # 達成率の振れ幅
            actual = int(round((target * factor) / 10) * 10) # 実績時間
            
            data.append({
                '生徒': student,
                '日程': day,
                '科目': subj,
                '分野': topic,
                '目標時間': target,
                '実績時間': actual
            })

df = pd.DataFrame(data)

# ---------------------------------------------------------
# 3. 資産ランキング アルゴリズム
# ---------------------------------------------------------
# 達成率 ＝ 実績時間 / 目標時間
df['達成率'] = df['実績時間'] / df['目標時間']

# 資産(万円) ＝ 実績時間 × 達成率
df['資産(万円)'] = df['実績時間'] * df['達成率']

# ユーザーごとの合計資産を集計・ソート
ranking_df = df.groupby('生徒')['資産(万円)'].sum().reset_index()
ranking_df = ranking_df.sort_values(by='資産(万円)', ascending=False).reset_index(drop=True)

# 順位付け
ranking_df['順位'] = ranking_df.index + 1
ranking_df['資産(万円)'] = ranking_df['資産(万円)'].round(1) # 小数点第1位までに丸める

print("========================================")
print(" 🏆 資産ランキング（7日間の合計）")
print("========================================")
print(ranking_df[['順位', '生徒', '資産(万円)']].to_string(index=False))
print("\n")

# ---------------------------------------------------------
# 4. 分野別ヒートマップ 可視化処理
# ---------------------------------------------------------
# 日程 × 分野 ごとのユニーク学習者数（人数）を集計するピボットテーブル作成
heatmap_data = df.groupby(['分野', '日程'])['生徒'].nunique().unstack(fill_value=0)

# 表示順を6分野の指定順に揃える
topic_order = [t[1] for t in topics_6]
heatmap_data = heatmap_data.reindex(topic_order)

# グラフ描画
plt.figure(figsize=(10, 5))
sns.heatmap(
    heatmap_data, 
    annot=True,            # マスの中に人数を表示
    cmap='YlGnBu',         # 色のグラデーション（黄→緑→青）
    fmt='d',               # 整数表示
    linewidths=0.5,        # 枠線
    cbar_kws={'label': '学習者数（人）'}
)

plt.title('【学習トレンド】分野別・日別 アクティブ学習者数ヒートマップ', fontsize=14, pad=15)
plt.xlabel('日程', fontsize=12)
plt.ylabel('学習分野（6分野）', fontsize=12)
plt.tight_layout()
plt.show()
