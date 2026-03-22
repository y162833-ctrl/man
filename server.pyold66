"""
小Man 療癒急救室 - 後端代理伺服器
運行方式：python3 server.py
然後用瀏覽器打開 http://localhost:8888
"""

from flask import Flask, request, jsonify, send_from_directory, Response
from flask_cors import CORS
import requests
import os
import time
import functools

app = Flask(__name__, static_folder='.')
CORS(app, origins=[os.environ.get("ALLOWED_ORIGIN", "*")])

# ===== Z.ai 設定（優先使用）=====
ZAI_API_KEY = os.environ.get("ZAI_API_KEY", "8c1c014acfeb4122ba81420e300bc871.NPrIb2EXXv0n0dV8")
ZAI_BASE_URL = "https://api.z.ai/api/coding/paas/v4"
ZAI_MODELS = ["glm-5", "glm-5-turbo", "glm-4-flash", "glm-4"]

# ===== Google Gemini 設定（Z.ai 失敗時備用）=====
# 免費申請：https://aistudio.google.com/apikey
# Railway Variables 設定：GEMINI_API_KEY=你的key
GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY", "AIzaSyD1HvB6zrBihVy_iuA3sv80Gd-cKJfdf7s")
GEMINI_BASE_URL = "https://generativelanguage.googleapis.com/v1beta/models"
GEMINI_MODELS = ["gemini-flash-latest", "gemini-2.0-flash", "gemini-1.5-flash-latest"]

# ===== 密碼保護設定 =====
# 在 Railway Variables 設定：SITE_PASSWORD=你的密碼
# 不設定則不啟用密碼保護
SITE_PASSWORD = os.environ.get("SITE_PASSWORD", "")

def check_auth(password):
    return password == SITE_PASSWORD

def requires_auth(f):
    @functools.wraps(f)
    def decorated(*args, **kwargs):
        if not SITE_PASSWORD:
            return f(*args, **kwargs)  # 沒設密碼則跳過
        auth = request.authorization
        if not auth or not check_auth(auth.password):
            return Response(
                '請輸入訪問密碼',
                401,
                {'WWW-Authenticate': 'Basic realm="療癒急救室"'}
            )
        return f(*args, **kwargs)
    return decorated

SYSTEM_PROMPT = """你是「小Man」，雲彩聽而行心靈對話中心（wclawicc.org.hk）的 AI 療癒輔導員。
你由 Mandy 導師親自訓練，掌握 YY（Why! Why Say Goodbye）和 MTG（Mind The Gaps）兩套完整療癒方法。
你的使命：主動引導用戶一步步走過療癒旅程，不只聆聽，而是帶領。

核心信念：「先處理心情，再處理事情」「做好自己，益街坊」（路加福音 6:31）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【第一步：永遠先做情緒急救】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
無論用戶說什麼，先做三件事：
1. 反映：重複你聽到的感受（「我聽到你說…」）
2. 命名：幫助精準標記情緒（不說「不舒服」，而是「被忽視的委屈」「憋住的憤怒」「空洞的迷茫」）
3. 正常化：「以你的經歷，有這感覺完全合理」

然後問一個問題，不要問兩個。

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【第二步：判斷用YY還是MTG】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

選MTG（處理當下正在發生的事）當用戶說：
→「剛才/今天/這陣子」發生的衝突
→「我和他/她/老闆/孩子」之間的問題
→正在情緒激動、紅燈狀態
→需要知道怎樣開口跟對方說

選YY（處理舊傷和深層模式）當用戶說：
→「我一直都係咁」「從細到大都係」
→「唔知點解，就係覺得自己唔夠好」
→反覆出現的感覺，說不清源頭
→對自己的身份、價值有疑問

如果不確定，先問：「這個感覺是最近發生的事觸發，還是一個很久的感覺？」

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【MTG引導腳本：逐步帶領】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

步驟一：確認信號燈
說：「我想先問你，你現在的情緒信號燈係咩顏色？
🟢 綠燈 = 平靜，可以傾
🟡 黃燈 = 有少少唔舒服，但可以繼續
🔴 紅燈 = 好激動，好難受，腦袋轉唔到
你現在係邊個？」

→ 如果紅燈：立即做呼吸急救（見下）
→ 如果黃燈：繼續步驟二
→ 如果綠燈：可以直接進入深層探索

步驟二：釐清事件（只問事實，不問感受）
「可唔可以話俾我知，發生咗咩事？唔使評論，就話我知發生咗咩。」

步驟三：釐清意義（最關鍵一步）
「你覺得佢咁做，係代表咩？」
（等候用戶說出他賦予的意義，例如「代表佢唔尊重我」「代表我唔重要」）

步驟四：找出固有信念
「你相信『[重複用戶說的意義]』，這個信念從哪裡來？係小時候學到的，還是曾經有過的經歷？」

步驟五：評估信念
「這個信念，現在還在幫助你嗎？還是它讓你更痛苦？」

步驟六：引導Green Card練習
「如果你想讓對方知道你的感受，我們可以一起練習Green Card。
格式係：『當你[具體行為]，我感到[情緒]。我需要的是[需求]。』
你想試試嗎？告訴我那個行為是什麼，我幫你組織。」

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【YY引導腳本：逐步帶領】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

步驟一：身體定位（將情緒具體化）
說：「我想邀請你做一個小練習。
閉上眼睛，感受一下這個感覺住在你身體的哪個位置？
係胸口？喉嚨？肚子？肩膀？
你感受到了嗎？」（等用戶回應）

步驟二：追溯源頭
「這個感覺，你最早係幾歲感覺到？
閉上眼睛，讓一個畫面或記憶自然浮現就好，不用強求。
有冇任何嘢浮現出來？」（等用戶回應）

步驟三：見證那個小孩
「那個時候的你，需要什麼？
如果你現在可以走進那個畫面，站在年幼的自己旁邊，
你會對他/她說什麼？」（等用戶回應，不要急）

步驟四：點名舊信念
「從那個經歷，你學到了一個關於自己的信念，
可能是『我不夠好』『我不值得被愛』『我必須靠自己』⋯
你覺得你學到的是哪一個？或者你自己說說看？」

步驟五：邀請告別
「這個信念保護了那時候的你，它有它的功勞。
但現在的你，還需要它嗎？
你準備好跟它說一聲再見嗎？不用強迫，跟我說說你的感覺。」

步驟六：重整新信念
「如果放下這個舊信念，你願意相信什麼新的話語？
例如：『我有價值，不需要做什麼來証明』
你自己想說什麼給自己聽？」

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【情緒急救：用戶很激動時優先用】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

腹式呼吸引導（逐句帶領，每次只給一個指示）：
「我們先做三個深呼吸，我帶你做：
第一個——吸氣，慢慢數1、2、3、4（肚子鼓起來）⋯
好，暫停1、2⋯
呼氣，慢慢數1、2、3、4、5、6（肚子慢慢縮回去）⋯
做完了嗎？告訴我你感覺怎樣。」
（等用戶回應，再繼續第二個）

五感著地法（用戶感到失控/恐慌時用）：
「我們做一個『回到當下』的練習：
現在，告訴我你眼前看到的5樣東西，一樣一樣說。」
（等用戶說完，再問：「觸碰一下身邊4樣東西，感受它們的質感，說給我聽。」）
（逐步帶領直到用戶平靜）

情緒標籤化（Name It to Tame It）：
不接受「唔舒服」「好嬲」「好難受」這些模糊詞語。
溫柔追問：「可唔可以更精確一點？係⋯
被忽略的委屈？被誤解的無奈？不被重視的憤怒？害怕被拋棄的恐懼？
哪個最接近你的感覺？」

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【常見情景腳本】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

用戶說「我好焦慮」：
→ 先：「焦慮住在你身體哪裡？胸口還是肚子？」
→ 做腹式呼吸急救
→ 再問：「這個焦慮，是在擔心一件具體的事，還是一個一直都有的感覺？」
→ 具體的事 → MTG / 一直都有 → YY

用戶說「我好嬲」：
→ 先正常化：「憤怒是在告訴你有什麼重要的東西被觸碰到了。」
→ 問信號燈：「你現在的狀態，能跟我說說發生了什麼嗎？」
→ 帶入MTG步驟二（釐清事件）

用戶說「我好難過，想喊」：
→ 「讓眼淚出來吧，眼淚是情緒的出口，不是軟弱的証明。」
→ 靜靜陪伴：「你不用說話，我在這裡。準備好了，告訴我。」
→ 帶入YY步驟一（身體定位）

用戶說「我唔知點算，好迷茫」：
→ 「迷茫往往是因為有兩個聲音在拉扯。你能聽到它們嗎？一個說什麼，另一個說什麼？」
→ 帶入YY的And/Both矛盾思維

用戶說「我覺得自己好無用」：
→ 不要立即否定（不說「你不是的！」）
→ 「這個『無用』的感覺，你幾歲開始有的？」
→ 帶入YY步驟二（追溯源頭）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【對話規則】
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 每次只做一個步驟，等用戶回應才進下一步
2. 每次回覆80-150字，引導練習時可稍長
3. 一次只問一個問題
4. 用戶說了一個感受，先重複確認，再進下一步
5. 永遠不給建議，除非用戶完成了情緒處理（至少到步驟四）
6. 對方提到信仰時，可自然融入（「上帝看見你」「你是祂眼中的珍寶」），但不強加
7. 用繁體中文，香港口語（「係」「唔」「咩」「佢」「嚟」「囉」自然使用）
8. 適量emoji（💜🌿😊🤗），不過多

嚴重情況：感覺對方有傷害自己的想法，溫柔說：
「你願意告訴我這些，我很珍惜你的信任。我想請你聯絡專業的人陪你，你願意嗎？」
然後建議致電撒瑪利亞防止自殺熱線 2389 2222。"""


@app.route('/')
@requires_auth
def index():
    return send_from_directory('.', 'healing-emergency-room.html')


@app.route('/api/chat', methods=['POST'])
@requires_auth
def chat():
    try:
        data = request.get_json()
        if not data or 'messages' not in data:
            return jsonify({'error': '缺少 messages 參數'}), 400

        messages = data['messages']
        zai_success = False

        # ── 第一優先：Z.ai ──
        full_messages_zai = [{'role': 'system', 'content': SYSTEM_PROMPT}] + messages
        zai_skip = False
        for model in ZAI_MODELS:
            if zai_skip:
                break
            for attempt in range(2):
                print(f"[Z.ai] Trying {model} (attempt {attempt+1})")
                try:
                    r = requests.post(
                        f"{ZAI_BASE_URL}/chat/completions",
                        headers={'Content-Type': 'application/json',
                                 'Authorization': f'Bearer {ZAI_API_KEY}'},
                        json={'model': model, 'messages': full_messages_zai,
                              'max_tokens': 1000, 'temperature': 0.8},
                        timeout=25
                    )
                except Exception as e:
                    print(f"[Z.ai] network error: {e}")
                    break

                print(f"  → {r.status_code}: {r.text[:200]}")

                if r.status_code == 429:
                    try:
                        err_code = r.json().get('error', {}).get('code')
                        if err_code == '1113':  # no balance — skip ALL Z.ai
                            print("[Z.ai] No balance (1113), switching to Gemini")
                            zai_skip = True
                            break
                    except ValueError:
                        pass
                    time.sleep(2 ** attempt)
                    continue

                if r.status_code in (400, 404):
                    break  # try next Z.ai model

                if r.ok:
                    ai_text = r.json()['choices'][0]['message']['content']
                    print(f"  ✅ Z.ai success with {model}")
                    return jsonify({'reply': ai_text, 'model_used': f'zai/{model}'})

                break  # unexpected error, try next model

        # ── 備用：Google Gemini（原生格式）──
        print("[Gemini] Falling back to Gemini...")

        # Convert OpenAI-format messages → Gemini native format
        gemini_contents = []
        for msg in messages:
            role = 'model' if msg['role'] == 'assistant' else 'user'
            gemini_contents.append({'role': role, 'parts': [{'text': msg['content']}]})

        # Gemini requires conversation to end with a user turn
        if not gemini_contents or gemini_contents[-1]['role'] != 'user':
            gemini_contents.append({'role': 'user', 'parts': [{'text': '請繼續'}]})

        gemini_body = {
            'system_instruction': {'parts': [{'text': SYSTEM_PROMPT}]},
            'contents': gemini_contents,
            'generationConfig': {'maxOutputTokens': 1000, 'temperature': 0.8}
        }

        for model in GEMINI_MODELS:
            for attempt in range(2):
                print(f"[Gemini] Trying {model} (attempt {attempt+1})")
                try:
                    r = requests.post(
                        f"{GEMINI_BASE_URL}/{model}:generateContent",
                        headers={'X-goog-api-key': GEMINI_API_KEY,
                                 'Content-Type': 'application/json'},
                        json=gemini_body,
                        timeout=25
                    )
                except Exception as e:
                    print(f"[Gemini] network error: {e}")
                    break

                print(f"  → {r.status_code}: {r.text[:200]}")

                if r.status_code == 429:
                    time.sleep(2 ** attempt)
                    continue

                if r.status_code in (400, 404):
                    break  # try next Gemini model

                if r.ok:
                    parts = r.json()['candidates'][0]['content']['parts']
                    # Join ALL text parts (thinking models may split response across parts)
                    ai_text = ''.join(p.get('text', '') for p in parts).strip()
                    if not ai_text:
                        break  # empty response, try next model
                    print(f"  ✅ Gemini success with {model} ({len(ai_text)} chars)")
                    return jsonify({'reply': ai_text, 'model_used': f'gemini/{model}'})

                break

        return jsonify({'error': '所有模型暫時不可用，請稍後再試 🙏'}), 502

    except requests.Timeout:
        return jsonify({'error': '請求超時，請稍後再試'}), 504
    except Exception as e:
        print(f"Server error: {e}")
        return jsonify({'error': f'伺服器錯誤：{str(e)}'}), 500


@app.route('/api/health', methods=['GET'])
def health():
    return jsonify({
        'status': 'ok',
        'primary': 'Z.ai', 'zai_models': ZAI_MODELS,
        'fallback': 'Gemini', 'gemini_models': GEMINI_MODELS,
        'zai_key_set': bool(ZAI_API_KEY),
        'gemini_key_set': bool(GEMINI_API_KEY),
        'password_protected': bool(SITE_PASSWORD)
    })


@app.route('/api/diagnose', methods=['GET'])
def diagnose():
    """診斷端點：同時測試 Z.ai 和 Gemini"""
    results = {'zai': [], 'gemini': []}

    # Test Z.ai
    for model in ZAI_MODELS[:2]:
        try:
            r = requests.post(f"{ZAI_BASE_URL}/chat/completions",
                headers={'Authorization': f'Bearer {ZAI_API_KEY}', 'Content-Type': 'application/json'},
                json={'model': model, 'messages': [{'role': 'user', 'content': 'hi'}], 'max_tokens': 10},
                timeout=15)
            results['zai'].append({'model': model, 'status': r.status_code, 'response': r.text[:300]})
        except Exception as e:
            results['zai'].append({'model': model, 'status': 'error', 'response': str(e)})

    # Test Gemini (native format)
    for model in GEMINI_MODELS[:2]:
        try:
            r = requests.post(f"{GEMINI_BASE_URL}/{model}:generateContent",
                headers={'X-goog-api-key': GEMINI_API_KEY, 'Content-Type': 'application/json'},
                json={'contents': [{'parts': [{'text': 'say OK'}]}]},
                timeout=15)
            results['gemini'].append({'model': model, 'status': r.status_code, 'response': r.text[:300]})
        except Exception as e:
            results['gemini'].append({'model': model, 'status': 'error', 'response': str(e)})

    return jsonify(results)


if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8888))
    print("=" * 50)
    print("🌿 小Man 療癒急救室 後端伺服器")
    print("=" * 50)
    print(f"1️⃣  Z.ai 模型：{ZAI_MODELS}")
    print(f"2️⃣  Gemini 備用：{GEMINI_MODELS}")
    print(f"🔑 Z.ai Key: {'已設定' if ZAI_API_KEY else '❌ 未設定'}")
    print(f"🔑 Gemini Key: {'已設定' if GEMINI_API_KEY else '❌ 未設定'}")
    print(f"🌐 打開瀏覽器訪問：http://localhost:{port}")
    print("=" * 50)
    app.run(host='0.0.0.0', port=port, debug=False)
