// Vercel Serverless Function: /api/spo
// Set GAS_WEB_APP_URL and GAS_PROXY_SECRET in Vercel Project Settings > Environment Variables.

module.exports = async function handler(req, res) {
  res.setHeader('Content-Type', 'application/json; charset=utf-8');
  res.setHeader('Cache-Control', 'no-store');

  if (req.method !== 'POST') {
    res.setHeader('Allow', 'POST');
    return res.status(405).json({ ok: false, error: 'Method not allowed.' });
  }

  const gasUrl = process.env.GAS_WEB_APP_URL;
  const proxySecret = process.env.GAS_PROXY_SECRET;

  if (!gasUrl || !proxySecret) {
    return res.status(500).json({
      ok: false,
      error: 'Vercel chưa cấu hình GAS_WEB_APP_URL hoặc GAS_PROXY_SECRET.'
    });
  }

  try {
    const body = req.body || {};
    const action = String(body.action || '').trim();
    const args = Array.isArray(body.args) ? body.args : [];

    // Allowlist ở Vercel là lớp lọc đầu tiên; Apps Script cũng có allowlist riêng.
    const allowedActions = new Set([
      'deleteServerSessionToken',
      'updateServerSessionMode',
      'restoreSessionUser',
      'checkZaloRegisterCode',
      'registerNewAccount',
      'getWithdrawDashboardData',
      'saveUserSecurityEmail',
      'saveMemberBankAccount',
      'sendWithdrawOtpEmail',
      'confirmWithdrawRequest',
      'loginUser',
      'CONVERT_LINK_DETAIL',
      'getOrdersAndTurns',
      'getMiniGameConfig',
      'submitMultiPredictions',
      'getHtmsSpoOrders',
      'getUserAccountDetails',
      'updateNickname',
      'changePassword'
    ]);

    if (!allowedActions.has(action)) {
      return res.status(400).json({ ok: false, error: 'Chức năng không được hỗ trợ.' });
    }
    if (args.length > 8) {
      return res.status(400).json({ ok: false, error: 'Quá nhiều tham số.' });
    }

    const upstream = await fetch(gasUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      redirect: 'follow',
      body: JSON.stringify({
        action: action,
        args: args,
        secret: proxySecret
      })
    });

    const responseText = await upstream.text();
    let data;
    try {
      data = JSON.parse(responseText);
    } catch (_parseError) {
      throw new Error('Apps Script trả về dữ liệu không phải JSON.');
    }

    if (!upstream.ok || !data || data.ok !== true) {
      return res.status(502).json({
        ok: false,
        error: (data && data.error) || ('Apps Script HTTP ' + upstream.status)
      });
    }

    return res.status(200).json({ ok: true, result: data.result });
  } catch (error) {
    return res.status(502).json({
      ok: false,
      error: error && error.message ? error.message : 'Không thể kết nối Apps Script.'
    });
  }
};
