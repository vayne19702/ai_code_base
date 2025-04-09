import pandas as pd
import lightgbm as lgb
import os
import json

df = pd.read_excel("your_data.xlsx")
df['Date'] = pd.to_datetime(df['Date'])

# 增加更多时间特征
df['day'] = df['Date'].dt.day
df['month'] = df['Date'].dt.month
df['day_of_week'] = df['Date'].dt.dayofweek
df['day_of_year'] = df['Date'].dt.dayofyear
df['is_weekend'] = (df['day_of_week'] >= 5).astype(int)

os.makedirs("models", exist_ok=True)

for visitor_id, user_df in df.groupby("VISITOR_ID"):
    menu_counts = user_df["CLICKPATH"].nunique()

    # 只访问过一个菜单的用户，保存默认菜单
    if menu_counts == 1:
        default_menu = user_df["CLICKPATH"].iloc[0]
        with open(f"models/{visitor_id}_default_menu.json", "w", encoding="utf-8") as f:
            json.dump({"default_menu": default_menu}, f, ensure_ascii=False, indent=2)
        continue

    # 构建菜单ID映射
    menu_to_id = {menu: idx for idx, menu in enumerate(user_df["CLICKPATH"].unique())}
    id_to_menu = {idx: menu for menu, idx in menu_to_id.items()}
    y = user_df["CLICKPATH"].map(menu_to_id)
    
    features = ['day', 'month', 'day_of_week', 'day_of_year', 'is_weekend']
    X = user_df[features]
    weights = user_df["PageViews"]

    train_data = lgb.Dataset(X, label=y, weight=weights)

    params = {
        'objective': 'multiclass',
        'num_class': len(menu_to_id),
        'metric': 'multi_logloss',
        'verbosity': -1,
        'learning_rate': 0.05,
        'num_leaves': 64,
        'max_depth': 7,
        'min_data_in_leaf': 3,
        'feature_pre_filter': False
    }

    model = lgb.train(params, train_data, num_boost_round=100)

    model.save_model(f"models/{visitor_id}_model.txt")

    with open(f"models/{visitor_id}_menu_map.json", "w", encoding="utf-8") as f:
        json.dump(id_to_menu, f, ensure_ascii=False, indent=2)




def predict_top3_menus(visitor_id, date_str):
    import lightgbm as lgb
    import pandas as pd
    import json
    import os

    model_path = f"models/{visitor_id}_model.txt"
    menu_map_path = f"models/{visitor_id}_menu_map.json"
    default_menu_path = f"models/{visitor_id}_default_menu.json"

    # 如果是只有一个菜单的用户
    if os.path.exists(default_menu_path):
        with open(default_menu_path, encoding="utf-8") as f:
            data = json.load(f)
        return [data["default_menu"]]

    if not os.path.exists(model_path):
        return []

    model = lgb.Booster(model_file=model_path)
    with open(menu_map_path, encoding="utf-8") as f:
        id_to_menu = json.load(f)

    # 构造日期特征
    date = pd.to_datetime(date_str)
    feature_row = pd.DataFrame([{
        'day': date.day,
        'month': date.month,
        'day_of_week': date.dayofweek,
        'day_of_year': date.dayofyear,
        'is_weekend': int(date.dayofweek >= 5)
    }])

    probs = model.predict(feature_row)[0]
    top3_indices = sorted(range(len(probs)), key=lambda i: probs[i], reverse=True)[:3]
    top3_menus = [id_to_menu[str(i)] for i in top3_indices]

    return top3_menus

