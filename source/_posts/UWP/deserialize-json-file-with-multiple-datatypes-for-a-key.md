---
title: Json一个key有多种数据类型情况的处理
tags:
  - Json
  - Newtonsoft
categories:
  - - 开发笔记
date: 2020-03-07 03:01:56
---

描述的语言，暂且搁置。备忘。

{
"itemList": \[{
"data": {
"cover": {
"feed": "http://img.kaiyanapp.com/5186fe44e2e90bb0013cc86b9808b710.png?imageMogr2/quality/60/format/jpg",
"detail": "http://img.kaiyanapp.com/5186fe44e2e90bb0013cc86b9808b710.png?imageMogr2/quality/60/format/jpg",
"blurred": "http://img.kaiyanapp.com/81fc3c98ac4dcbc6c955cf3b1805f051.jpeg?imageMogr2/quality/60/format/jpg",
"sharing": null,
"homepage": null
}
}
},
{
"data": {
"cover": "http://img.kaiyanapp.com/3bc8837d3320684bbc3607b82ff94761.jpeg?imageMogr2/quality/60/format/jpg"
}
}
\]
}

```
 [JsonConverter(typeof(CoverConverter))]
 public Cover cover { get; set; }
```

public class CoverConverter : JsonConverter
    {
        public override bool CanConvert(Type objectType) => false;

        private Cover ParseValue(JToken value)
        {
            switch (value.Type)
            {
                case JTokenType.Object:
                    return value.ToObject<Cover>();

                default:
                    return null;
            }
        }

        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            switch (reader.TokenType)
            {
                case JsonToken.StartObject:
                    return ParseValue(JToken.Load(reader));

                default:
                    return null;
            }
        }

        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            throw new NotImplementedException();
        }
    }

参考文章：[Deserialize JSON-File with multiple datatypes for a key](https://stackoverflow.com/questions/56206243/deserialize-json-file-with-multiple-datatypes-for-a-key)