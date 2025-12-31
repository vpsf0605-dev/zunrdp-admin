<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ZUNRDP | QUẢN TRỊ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background-color: #020408;
            color: #f8fafc;
            -webkit-tap-highlight-color: transparent;
        }

        .bg-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: radial-gradient(circle at 50% 0%, #1e293b 0%, #020408 100%);
            z-index: -1;
        }

        .zun-logo {
            font-weight: 800; font-size: 24px; letter-spacing: -1px;
            background: linear-gradient(to right, #ffffff, #3b82f6);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
        }
        .zun-logo span { color: #3b82f6; -webkit-text-fill-color: #3b82f6; }

        .rdp-card {
            background: rgba(15, 23, 42, 0.7);
            backdrop-filter: blur(25px);
            -webkit-backdrop-filter: blur(25px);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 32px;
            padding: 24px;
            margin-bottom: 20px;
        }

        .btn-action {
            height: 50px; border-radius: 16px; font-weight: 700; font-size: 13px;
            display: flex; align-items: center; justify-content: center; gap: 8px;
            transition: all 0.2s;
            width: 100%;
        }
        .btn-action:active { transform: scale(0.96); opacity: 0.8; }

        .btn-primary { background: #fff; color: #000; }
        .btn-danger { background: rgba(239, 68, 68, 0.15); color: #ef4444; border: 1px solid rgba(239, 68, 68, 0.2); }

        /* Nút copy nhỏ tinh tế */
        .copy-btn {
            font-size: 10px; background: rgba(59, 130, 246, 0.1); color: #60a5fa;
            padding: 4px 10px; border-radius: 8px; border: 1px solid rgba(59, 130, 246, 0.2);
            cursor: pointer; font-weight: 700; text-transform: uppercase;
        }
        .copy-btn:active { background: #3b82f6; color: #fff; transform: scale(0.9); }

        .mono { font-family: ui-monospace, SFMono-Regular, monospace; }

        .status-online {
            width: 8px; height: 8px; background: #10b981; border-radius: 50%;
            box-shadow: 0 0 10px #10b981;
        }
    </style>
</head>
<body class="p-5 pb-10">

    <div class="bg-overlay"></div>

    <div class="max-w-md mx-auto">
        <header class="flex justify-between items-start mt-4 mb-8">
            <div>
                <div class="zun-logo">ZUN<span>RDP</span></div>
                <p class="text-[9px] font-bold text-slate-500 uppercase tracking-[2px] mt-1 text-nowrap">Hệ Thống Cloud Việt Nam</p>
            </div>
            <div class="bg-white/5 px-4 py-2 rounded-2xl border border-white/5 text-center min-w-[80px]">
                <span id="node-count" class="text-xl font-bold text-blue-400">0</span>
                <p class="text-[7px] text-slate-500 font-black uppercase">Trực tuyến</p>
            </div>
        </header>

        <div id="vm-list" class="space-y-4">
            <div class="text-center py-10 opacity-20"><i class="fas fa-spinner fa-spin"></i></div>
        </div>
    </div>

    <div id="toast" class="fixed top-8 left-1/2 -translate-x-1/2 z-50 opacity-0 transition-all duration-300 pointer-events-none">
        <div class="bg-blue-600 text-white px-6 py-3 rounded-full font-bold text-xs shadow-2xl"></div>
    </div>

    <script>
        const API_URL = "https://zunrdp-default-rtdb.asia-southeast1.firebasedatabase.app";

        function copyToClipboard(text, label) {
            navigator.clipboard.writeText(text);
            const t = document.getElementById('toast');
            t.firstElementChild.innerText = `Đã chép ${label}`;
            t.classList.remove('opacity-0', '-translate-y-2');
            t.classList.add('opacity-100', 'translate-y-0');
            setTimeout(() => {
                t.classList.add('opacity-0', '-translate-y-2');
                t.classList.remove('opacity-100', 'translate-y-0');
            }, 1500);
        }

        async function refresh() {
            try {
                const res = await fetch(`${API_URL}/vms.json`);
                const data = await res.json();
                const container = document.getElementById('vm-list');
                let count = 0;
                let html = '';

                for (let id in data) {
                    const vm = data[id];
                    const diff = Math.floor((Date.now() - vm.lastSeen) / 1000);
                    if (diff > 60) continue;
                    count++;

                    const isWin = vm.os === 'Windows';

                    html += `
                    <div class="rdp-card">
                        <div class="flex justify-between items-start mb-6">
                            <div class="flex items-center gap-3">
                                <div class="status-online"></div>
                                <h3 class="font-bold text-lg tracking-tight">${vm.id}</h3>
                            </div>
                            <div class="text-right">
                                <p class="text-[9px] font-black text-slate-500 uppercase tracking-widest">${isWin ? 'Win Server 2025' : 'Ubuntu'}</p>
                                <p class="text-[8px] text-emerald-400 font-bold mt-1 italic">Cập nhật: ${diff}s trước</p>
                            </div>
                        </div>

                        <div class="grid grid-cols-2 gap-3 mb-6">
                            <div class="bg-white/5 p-3 rounded-2xl border border-white/5 text-center">
                                <p class="text-[8px] text-slate-500 uppercase font-bold mb-1">Cấu hình CPU</p>
                                <p class="text-xs font-bold mono">${isWin ? '4 Core vCPU' : '2 Core'}</p>
                            </div>
                            <div class="bg-white/5 p-3 rounded-2xl border border-white/5 text-center">
                                <p class="text-[8px] text-slate-500 uppercase font-bold mb-1">Bộ nhớ RAM</p>
                                <p class="text-xs font-bold mono">${isWin ? '16 GB RAM' : '8 GB'}</p>
                            </div>
                        </div>

                        <div class="bg-black/40 rounded-2xl p-4 mb-6 border border-white/5 space-y-4">
                            <div class="flex justify-between items-center">
                                <span class="text-[10px] text-slate-500 font-bold uppercase">Địa chỉ IP</span>
                                <div class="flex items-center gap-3">
                                    <span class="text-sm font-bold text-blue-400 mono">${vm.ip}</span>
                                    <div class="copy-btn" onclick="copyToClipboard('${vm.ip}', 'IP')">Sao chép</div>
                                </div>
                            </div>
                            <div class="space-y-3 pt-3 border-t border-white/10">
                                <div class="flex justify-between items-center">
                                    <span class="text-[11px] text-slate-300">Tài khoản: <b class="text-white ml-1 font-bold">ADMINZUN</b></span>
                                    <div class="copy-btn" onclick="copyToClipboard('ADMINZUN', 'Tài khoản')">Sao chép</div>
                                </div>
                                <div class="flex justify-between items-center">
                                    <span class="text-[11px] text-slate-300">Mật khẩu: <b class="text-blue-400 ml-1 font-bold mono">ZunRDP@123456</b></span>
                                    <div class="copy-btn" onclick="copyToClipboard('ZunRDP@123456', 'Mật khẩu')">Sao chép</div>
                                </div>
                            </div>
                        </div>

                        <div class="flex gap-3">
                            <button onclick="cmd('${id}', 'restart')" class="btn-action btn-primary">
                                <i class="fas fa-rotate"></i> Khởi động lại
                            </button>
                            <button onclick="cmd('${id}', 'kill')" class="btn-action btn-danger">
                                <i class="fas fa-power-off"></i> Ngắt máy
                            </button>
                        </div>
                    </div>`;
                }
                container.innerHTML = html;
                document.getElementById('node-count').innerText = count;
            } catch (e) {}
        }

        async function cmd(id, type) {
            if(!confirm(`Xác nhận thực hiện lệnh trên máy ${id}?`)) return;
            await fetch(`${API_URL}/commands/${id}.json`, { method: 'PUT', body: JSON.stringify(type) });
            copyToClipboard('', 'Yêu cầu');
        }

        setInterval(refresh, 5000);
        refresh();
    </script>
</body>
</html>
