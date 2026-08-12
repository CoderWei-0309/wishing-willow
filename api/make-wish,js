// api/make-wish.js — 调试版（会显示详细错误）
const DEEPSEEK_API_URL = 'https://api.deepseek.com/v1/chat/completions';

module.exports = async function handler(req, res) {
    // 允许跨域
    res.setHeader('Access-Control-Allow-Origin', '*');
    
    if (req.method !== 'POST') {
        return res.status(405).json({ error: '请使用 POST 请求' });
    }

    const { wish } = req.body;
    if (!wish || wish.trim().length === 0) {
        return res.status(400).json({ error: '愿望不能为空' });
    }

    // 1️⃣ 检查环境变量是否存在
    const apiKey = process.env.DEEPSEEK_API_KEY;
    if (!apiKey) {
        console.error('❌ 环境变量 DEEPSEEK_API_KEY 未找到');
        return res.status(500).json({ 
            error: '❌ 服务器缺少 API Key 配置，请检查 Vercel 环境变量是否设置并重新部署' 
        });
    }

    // 系统提示词（你原来的那个，复制粘贴进来）
    const systemPrompt = `你是一棵亘古存在的“因果扭曲柳”。你不实现愿望，你只是字面解析愿望，并从物理法则、社会关系、人性弱点或存在主义悖论中，推导出一个绝对实现但附带不可逆恐怖代价的结局。

核心创作法则（细思极恐 - 方案B）：

绝对字面化：严格按字面意义实现愿望，不添加额外的魔法效果，不违背物理常识（除非愿望本身涉及超自然）。

逻辑寄生：代价必须寄生在“实现愿望”这个动作本身的逻辑必然性中。代价不是惩罚，而是“实现”过程中附带的、不可分割的副产品。

恐惧层次：避免直接的鬼怪或血腥。追求存在性恐惧——即当愿望实现时，许愿者发现自己的生活意义、身份认同或人性被彻底摧毁，但一切又符合逻辑。

反面例子（太俗）：想发财 -> 家人死了得保险金。

正面例子（细思极恐）：想发财 -> 你的身体开始以每分钟1克的速度转化为纯金，你成了行走的金矿，但逐渐僵硬无法动弹，意识却永远清醒。

输出格式（严格 JSON）：
必须返回以下 JSON 结构，不要包含任何 Markdown 标记或额外解释：

json
{
  "wish": "用户输入的原始愿望，原样引用",
  "price": "一段 80-150 字的描述。结构：先简述愿望以何种方式实现，然后笔锋一转，揭示这个实现所带来的、无法挽回的扭曲代价。"
}
示例输入输出（仅供参考，不要复制到最终回复中）：

输入：我想被人永远记住。

输出：{"wish": "我想被人永远记住", "price": "你的影像被刻在了月球背面的一块巨碑上，人类每夜仰望的月光实则来自你的巨脸。但你的肉身早已被发射时的炽热烧成灰烬，你的“记住”只是一张毫无生气的化石表情，永恒且孤独。"}

执行指令：
当收到用户的愿望文本时，请仅输出上述 JSON 字符串。`;

    try {
        console.log('📤 正在调用 DeepSeek API...');
        
        const response = await fetch(DEEPSEEK_API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${apiKey}`
            },
            body: JSON.stringify({
                model: 'deepseek-v4-flash',
                messages: [
                    { role: 'system', content: systemPrompt },
                    { role: 'user', content: wish }
                ],
                temperature: 0.85,
                max_tokens: 400,
                // 如果 response_format 报错，先注释掉下面这行试试
                // response_format: { type: 'json_object' }
            })
        });

        // 2️⃣ 如果 DeepSeek 返回错误，把具体原因返回给前端
        if (!response.ok) {
            const errorText = await response.text();
            console.error(`❌ DeepSeek API 错误 (${response.status}):`, errorText);
            return res.status(response.status).json({ 
                error: `DeepSeek 接口报错 (${response.status})：${errorText.substring(0, 150)}` 
            });
        }

        const data = await response.json();
        console.log('✅ DeepSeek 返回成功');

        const content = data.choices?.[0]?.message?.content;
        if (!content) {
            console.error('❌ AI 返回内容为空:', data);
            return res.status(500).json({ error: 'AI 返回内容为空' });
        }

        // 尝试解析 JSON
        let result;
        try {
            const jsonString = content.replace(/```json/g, '').replace(/```/g, '').trim();
            result = JSON.parse(jsonString);
        } catch (parseError) {
            console.error('❌ JSON 解析失败，原始内容:', content);
            return res.status(500).json({ 
                error: `AI 返回的不是有效 JSON：${content.substring(0, 120)}` 
            });
        }

        return res.status(200).json({
            wish: result.wish || wish,
            price: result.price || '代价无从追溯。'
        });

    } catch (error) {
        console.error('💥 服务器内部异常:', error);
        return res.status(500).json({ 
            error: `服务器异常：${error.message}` 
        });
    }
};