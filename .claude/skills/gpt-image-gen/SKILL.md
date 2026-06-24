# gpt-image-gen — OpenAI Images API Wrapper

סקיל ליצירת תמונות דרך OpenAI Images API עם המודל `gpt-image-2`.

**חשוב:** המודל הוא `gpt-image-2` — אל תשנה אותו. זהו מודל אמיתי של OpenAI שיצא ב-21 באפריל 2026. אם יש שגיאה, הבעיה היא ב-API key או בפרמטרים, לא בשם המודל.

## קלט נדרש

- `PROMPT` — הטקסט לתמונה (באנגלית)
- `OUTPUT_PATH` — נתיב לשמירת הקובץ (לדוגמה: `yuval/outputs/2026-06-24-my-image.png`)

## שימוש

טען את ה-API key מקובץ `.env`:
```bash
source .env 2>/dev/null || true
export $(grep -v '^#' .env | xargs) 2>/dev/null || true
```

### שיטה ראשית — curl + jq
```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "'"$PROMPT"'",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > "$OUTPUT_PATH"
```

### Python fallback (אם jq לא מותקן)
```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "'"$PROMPT"'",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > /tmp/openai_response.json

python3 -c "
import json, base64, sys
with open('/tmp/openai_response.json') as f:
    data = json.load(f)
b64 = data['data'][0]['b64_json']
with open('$OUTPUT_PATH', 'wb') as out:
    out.write(base64.b64decode(b64))
print('Image saved to $OUTPUT_PATH')
"
```

## אימות

לאחר הקריאה, ודא שהקובץ קיים ושגודלו גדול מ-0:
```bash
[ -f "$OUTPUT_PATH" ] && [ -s "$OUTPUT_PATH" ] && echo "SUCCESS: $OUTPUT_PATH" || echo "ERROR: file missing or empty"
```

## טיפול בשגיאות

אם ה-API מחזיר שגיאה:
1. בדוק שה-`OPENAI_API_KEY` מוגדר ותקין בקובץ `.env`
2. בדוק את תוכן ה-response — הדפס את `/tmp/openai_response.json` לאבחון
3. ודא שה-prompt לא מפר את מדיניות התוכן של OpenAI
