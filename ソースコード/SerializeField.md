* SerializedProperty.boxedValue 型を気にせずに使える
https://docs.unity3d.com/ja/2023.1/ScriptReference/SerializedProperty-boxedValue.html

型を気にせず、オブジェクト（構造体やクラス、基本型）を丸ごと一発で代入・取得出来ます
```C#
private object copiedData;
// コピー実行
private void CopyProperty(SerializedProperty prop)
{
    // どんな型（Int, Vector3, 自作構造体）であってもobjectとして保存できる
    copiedData = prop.boxedValue;
}

// ペースト実行
private void PasteProperty(SerializedProperty prop)
{
    if (copiedData != null)
    {
        // 型が一致していれば、そのまま流し込める
        prop.boxedValue = copiedData;
        prop.serializedObject.ApplyModifiedProperties();
    }
}
```

ログ出力も簡単になります
```C#
// どんな型プロパティであっても、一列のコードで現在の値を文字列化できる
Debug.Log($"現在の値: {prop.boxedValue.ToString()}");
```

機能制限
・Serializableが完全一致していなければエラーになります
・Int⇔floatの変換や変数名の不一致、継承の親子関係でもエラーが発生します
・汎用性を上げるためには型チェックが必要になります
```C#
private object copiedData;
private string copiedDataType; // コピー元の型名を保存

// コピー処理
private void CopyProperty(SerializedProperty prop)
{
    copiedData = prop.boxedValue;
    copiedDataType = prop.type; // 例: "int", "Vector3", "serializedStructure<MyData>" などが取れる
}

// ペースト処理
private void PasteProperty(SerializedProperty prop)
{
    if (copiedData == null) return;

    // 型名が完全に一致しているかチェック
    if (prop.type == copiedDataType)
    {
        prop.boxedValue = copiedData;
        prop.serializedObject.ApplyModifiedProperties();
        Debug.Log("ペーストに成功しました！");
    }
    else
    {
        // 型が違う場合は安全に無視、または警告を出す
        Debug.LogWarning($"型が不一致です。コピー元: {copiedDataType} / ペースト先: {prop.type}");
    }
}
```
