title: Intent常用操作
date: 2015-11-19 10:38:32
tags:
- Android
- Intent

---
### 跳转到市场评分

Uri uri = Uri.parse("market://details?id="+getPackageName());
Intent intent = new Intent(Intent.ACTION_VIEW,uri);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);

### 市场内搜索

Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("market://search?q=pub:Your Publisher Name"));
startActivity(intent);

### 分享

Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.setType("text/*");
sendIntent.putExtra(Intent.EXTRA_TEXT,str);
startActivity(sendIntent);