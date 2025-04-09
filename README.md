import pandas as pd
import lightgbm as lgb
import os

# 读取数据
df = pd.read_excel("user_clicks.xlsx")

# 日期特征
df['Date'] = pd.to_datetime(df['Date'])
df['day_of_week'] = df['Date'].dt.dayofweek  # 周几（0=Monday）
df['is_weekend'] = df['day_of_week'] >= 5    # 是否周末

# 模型保存目录
os.makedirs("models", exist_ok=True)

# 按用户训练模型或保存默认菜单
for user in df['VISITOR_ID'].unique():
    user_df = df[df['VISITOR_ID'] == user].copy()
    user_df['CLICKPATH'] = user_df['CLICKPATH'].astype('category')

    unique_menus = user_df['CLICKPATH'].nunique()

    if unique_menus < 2:
        # 只访问了一个菜单，保存默认菜单
        default_menu = user_df['CLICKPATH'].iloc[0]
        with open(f"models/{user}_default.txt", 'w', encoding='utf-8') as f:
            f.write(default_menu)
        print(f"用户 {user} 只访问过一个菜单，已保存默认菜单：{default_menu}")
        continue

    # 菜单编码
    label_map = dict(enumerate(user_df['CLICKPATH'].cat.categories))
    label_inverse_map = {v: k for k, v in label_map.items()}

    # 特征、标签、权重
    X = user_df[['day_of_week', 'is_weekend']]
    y = user_df['CLICKPATH'].cat.codes
    weights = user_df['PageViews']

    train_data = lgb.Dataset(X, label=y, weight=weights)

    params = {
        'objective': 'multiclass',
        'num_class': unique_menus,
        'metric': 'multi_logloss',
        'verbosity': -1
    }

    model = lgb.train(params, train_data, num_boost_round=100)

    # 保存模型和菜单映射
    model.save_model(f"models/{user}_model.txt")
    with open(f"models/{user}_labels.txt", 'w', encoding='utf-8') as f:
        for k, v in label_map.items():
            f.write(f"{k},{v}\n")
    print(f"已训练并保存用户 {user} 的模型")





def predict_top_k(user_id: str, date_str: str, k=3):
    default_path = f"models/{user_id}_default.txt"
    if os.path.exists(default_path):
        with open(default_path, 'r', encoding='utf-8') as f:
            default_menu = f.read().strip()
        print(f"用户 {user_id} 仅有一个菜单，返回默认菜单")
        return [default_menu] * min(k, 1)

    model_path = f"models/{user_id}_model.txt"
    label_path = f"models/{user_id}_labels.txt"

    if not os.path.exists(model_path) or not os.path.exists(label_path):
        print(f"用户 {user_id} 没有模型或标签映射文件")
        return []

    # 加载模型
    model = lgb.Booster(model_file=model_path)

    # 加载菜单映射
    label_map = {}
    with open(label_path, 'r', encoding='utf-8') as f:
        for line in f:
            k_, v_ = line.strip().split(',')
            label_map[int(k_)] = v_

    # 构造输入特征
    date = pd.to_datetime(date_str)
    day_of_week = date.dayofweek
    is_weekend = int(day_of_week >= 5)
    features = pd.DataFrame([[day_of_week, is_weekend]], columns=['day_of_week', 'is_weekend'])

    # 预测概率
    probs = model.predict(features)[0]
    top_k_indices = sorted(range(len(probs)), key=lambda i: probs[i], reverse=True)[:k]
    return [label_map[i] for i in top_k_indices]
