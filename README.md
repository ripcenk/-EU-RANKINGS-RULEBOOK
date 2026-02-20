<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Joud Core System - Blue Lock</title>
    <style>
        /* نظام الألوان والتنسيق الأسطوري الخاص بك */
        :root {
            --blue: #007bff;
            --dark-bg: #0a0a0a;
            --card-bg: #111;
            --text: #e0e0e0;
            --red: #ff4d4d;
            --green: #2ecc71;
        }

        body {
            background-color: var(--dark-bg);
            color: var(--text);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            overflow-x: hidden;
        }

        /* تنسيق الصور والقوانين */
        .rule-img { width: 100%; border-radius: 8px; margin: 10px 0; border: 1px solid var(--blue); }
        .bracket-text { color: var(--blue); font-weight: bold; }
        .word-banned { color: var(--red); font-weight: bold; text-transform: uppercase; }
        .word-allowed { color: var(--green); font-weight: bold; }
        .word-fg { color: #f1c40f; font-weight: bold; }
        
        /* الهيدر والقوائم */
        .header { background: #000; padding: 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--blue); }
        .nav-btn { background: none; border: 1px solid var(--blue); color: var(--blue); padding: 5px 15px; cursor: pointer; border-radius: 4px; }
        
        #main-menu { display: flex; flex-wrap: wrap; gap: 20px; padding: 20px; justify-content: center; }
        .cat-card { width: 150px; height: 150px; background: var(--card-bg); border: 1px solid #333; display: flex; flex-direction: column; align-items: center; justify-content: center; cursor: pointer; transition: 0.3s; border-radius: 12px; }
        .cat-card:hover { border-color: var(--blue); transform: translateY(-5px); box-shadow: 0 5px 15px rgba(0,123,255,0.3); }

        /* منطقة المحتوى */
        .page-view { display: none; padding: 20px; }
        .entry-item { background: #161616; margin-bottom: 10px; padding: 15px; border-radius: 8px; border-left: 4px solid var(--blue); }
        .entry-header { display: flex; justify-content: space-between; cursor: pointer; font-weight: bold; }
        .entry-content { display: none; margin-top: 10px; line-height: 1.6; }

        /* المودال والـ Admin */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 1000; justify-content: center; align-items: center; }
        .modal-content { background: #111; padding: 30px; border-radius: 15px; border: 1px solid var(--blue); width: 90%; max-width: 500px; }
        input, textarea { width: 100%; padding: 10px; margin: 10px 0; background: #222; border: 1px solid #444; color: white; border-radius: 5px; }
        
        .footer-spacer { height: 100px; display: flex; align-items: center; justify-content: center; opacity: 0.3; }
        .bl-footer { font-family: 'Arial Black'; font-size: 24px; letter-spacing: 5px; color: var(--blue); }
    </style>
</head>
<body>

    <div class="header">
        <button id="back-nav-btn" class="nav-btn" style="visibility: hidden;" onclick="closePage()">Back</button>
        <div id="user-profile" style="cursor:pointer" onclick="toggleProfileDropdown(event)">
            <span id="display-name">Guest</span>
        </div>
    </div>

    <div id="main-menu">
        <div class="cat-card" onclick="openPage('Style Rules')"><span>Style Rules</span></div>
        <div class="cat-card" onclick="openPage('Flow Rules')"><span>Flow Rules</span></div>
        <div class="cat-card" onclick="openPage('Global News & Archive')"><span>Global Archive</span></div>
    </div>

    <div id="page-view" class="page-view">
        <h2 id="page-title" style="color:var(--blue)"></h2>
        <div id="admin-panel" style="display:none">
            <textarea id="text-input" placeholder="Enter rule or Image URL..."></textarea>
            <button class="nav-btn" onclick="saveData()">Save Entry</button>
            <button class="nav-btn" style="color:var(--red); border-color:var(--red)" onclick="cancelEdit()">Cancel</button>
        </div>
        <input type="text" id="rule-search" placeholder="Search rules..." oninput="renderEntries()">
        <div id="data-list"></div>
    </div>

    <div id="backup-modal" class="modal">
        <div class="modal-content">
            <h3>System Backup</h3>
            <textarea id="backup-key-box" rows="5" readonly></textarea>
            <button class="nav-btn" onclick="restoreBackup()">Restore from Key</button>
            <button class="nav-btn" onclick="document.getElementById('backup-modal').style.display='none'">Close</button>
        </div>
    </div>

    <script>
        // كود الـ F7 الخاص بك (قاعدة البيانات المحقونة)
        const MASTER_BACKUP_KEY = "eyJkYiI6eyJTdHlsZSBSdWxlcyI6WyIjIExpbWl0ZWQgU3R5bGVzXG5cbktyYW1wb3MgQmFyb3UgQWxsb3dlZDpcbi1DcmFzaCBTdGVhbDsgQWxsb3dlZCBjb21wbGV0ZWx5LCAgaWxsZWdhbCB0byB1c2UgaW50byBvciBpbnNpZGUgeW91ciBvd24gYm94ICggY291bnQgYXMgYSBkZWZlbnNpdmUgbW92ZSksIGlmIHVzZWQgaW50byBvciBpbnNpZGUgdXIgb3duIGJveCwgcmVzdWx0cyBpbnRvIGEgRkcgaWYgaXQgc3RvcHBlZCBhIGdvYWwsIGJhbm5lZCB0byBzdGVhbCAtbW92ZSBmcm9tIG9wcG9uZW50IGJveFxuLUJsaXp6YXJkIEZhbmcgU2hvdDsgQWxsb3dlZCBhbnl3aGVyZSBleGNlcHQgaW5ib3guICBBbGxvd2VkIHdpdGggbWV0YXZpc2lvbiAvIHByZWRhdG9yIGthcmFtcHVzIGZyb20gc2Vjb25kIGxpbmUgLiAob2cga2Fpc2VyIHJhbmdlKVxuLUtyYW1wdXMgQ2hvcCBEcmliYmxlOyBiYW5uZWQgaW50by9pbiBib3ggY2Fubm90IHVzZSBzaG90IGFmdGVyIHVudGlsIHlvdSBsb3NlIHRoZSBiYWxsXG4tUHJlZGF0b3IgS3JhbXB1cy9NZXRhdmlzaW9uO0Nhbm5vdCB1c2Ugc3BlZWQgaW5jcmVhc2VkIGZsb3dzIHdpdGggaXQuIENhbm5vdCB1c2UgYmxpenphcmQgZmFuZyBzaG90IHdpdGggaXRcbi1pZiB1IGFpa3UgZm9yY2UgZGVmZW5zZSBrcmFtcHVzIGJhcm91IGFuZCBpdCBidWdzIGluIGdvYWwgYW55d2F5cyB0aGVuIGl0cyBhIGZyZWVnb2FsXG5cbkVsZiBFbXBlcm9yIEFsbG93ZWQ6XG4tRWxmIEltcGFjdDsgYWxsb3dlZCBhbnl3aGVyZSBleGNlcHQgaW5zaWRlIG9mIHRoZSBvcHBvbmVudOKAmXMgYm94XG4tSGVscGVyIERyaWJibGU7IEJBTk5FRCBpbnRvIGJveCwgYWxsb3dlZCB0byBiZSB1c2VkIHR3aWNlIC1FTEYgRU1QRVJPUiBWT0xMRVk7IGlzIGJhbm5lZFxuLUVsZiBJbXBhY3Qgd2hpbGUgcHJvZGlneSBpcyBvbiBpcyBiYW5uZWRcblxuU29iemVybyBMb2tpIEFsbG93ZWQ6XG4tfEFTIENNfFxuLUFpciB0cmFwIGluZiBvdCAocmVvIGdldHMgNCsxIGNhbW9zKSBhbmQgZHRyYXAgbmFnaSBiYW5uZWQgd2l0aCBpdFxuLUFpciB0cmFwIGJhbm5lZCB0byBnbG9iYWwgdHJhcFxuLUFpciB0cmFwIG5vdCBhbGxvd2VkIHRvIG1ha2UgTTFzIGdob3N0IG9uIGNoaWdpcmkgc3BlZWRzdGVyICh3aWxsIGNvdW50IGFzIHJiKVxuLUFpciB0cmFwIGFsbG93ZWQgdG8gbWFrZSBNMXMgZ2hvc3QgaW4gb3RoZXIgY2lyY3Vtc3RhbmNlcyAoY291bnRzIGFzIGEgdXNhZ2UpXG4tTmVsIFJlbyBhbGxvd2VkIHdpdGggaXQgYnV0IGlmIGJvdGggb24gdGhlIHBpdGNoIHdpdGggTG9raSBjbSB0aGVuIG5vIG90aGVyIG1ham9yXG5cblNhbnRhIExhdmluaG91IEFsbG93ZWQ6XG4tV1JBUFBFRCBEUklCQkw7IGJhbm5lZCBpbiBhbGxvd2VkIGludG8gYm94LSBcbi0gUFJFU0VOVCBTVFJJS0U7IGFsbG93ZWQgYW55d2hlcmUgYnV0IGJveCByZWFzb247IG5vdCBhcyBmYXN0IGFzIGtyYW1wdXMgYW5kIEthaSArIGNhbid0IGdyYXNzY3V0ICsgY2FuJ3QgYm91bmNlICsgbm8gaWZyYW1lcyBhZnRlciBzbyBVIGNhbiBnb2FsdGVuZCBpdC1cbi1TTEVJR0ggUklERTsgbm8gcmVzdHJpY3Rpb25zIChidW5zKVxuLUNIUklTVE1BUyBTVEFNUEVERTsgQWxsb3dlZCBas29sbHkgZGVmZW5zaXZlIHNpZGUsIGl0IGNvdW50cyBhcyBkZWZlbnNpdmUgbW92ZVxuLSBCYXJyaWVyIHNob3RzIGFyZSBiYW5uZWRcbi0gcGFzcyBtdXN0IGJlIG9ubHkgdXNlZCB3aGVuIHlvdSBoYXZlIHRoZSBiYWxsXG5cbiIsIiMgRXBpYyBTdHlsZXNcblxuS3Vyb25hIEFsbG93ZWQ6XG4tIG9uZS10d28gaW4vaW50byBiaWcgcGVuYWx0eSBib3ggaXMgYmFubmVkXG4tIFNoYXJrc3RlYWwgNSsxIGluIG90LCBvbmx5IHVyIHNpZGUgc3RyaWN0bHksIG1pc3NlZCBkb2VzbnRiY291bnRcbi0gb25lLXR3byAgdG8gYW55IHN0eWxlIHNob3QgbGlrZSBpbXBhY3QgaXMgYmFubmVkXG5cbk90b3lhIEFsbG93ZWQ6XG5cbkdhZ2FtYXJ1IEFsbG93ZWQ6XG4tIFBvcHBpbmcgRmFzdCByZWFjdGlvbnMgYW5kIEdvZHNwZWVkIHNhbWUgdGltZSBpcyBiYW5uZWRcblxuIiwiIyBSYXJlIFN0eWxlc1xuXG5Jc2FnaSBBbGxvd2VkOlxuXG5IaXJvcmkgQWxsb3dlZDpcbi0gVFAgc2hvdCB3aXRoIGxhc2VyIGlzIGJhbm5lZFxuXG5DaGlncmkgQWxsb3dlZDpcbi0gQWNjZWxlcmF0ZSBpbnRvIC8gaW4gbWluaWJveCBpcyBiYW5uZWRcbi1JZmFndXJpIEFsbG93ZWQ6XG4tIE1hbGljaWE7IG9ubHkgYWxsb3dlZCBpbiB1ciBvd24gaGFsZlxuLSBBaXIgTWFsaWNpYSBhbGxvd2VkIGluIG9wcG9uZW50IGhhbGYgb25seSB3aGVuIHUgaGF2ZSBiYWxsIHBvc3Nlc3Npb24gLSBhcyBwYXNzXG4tIEF1dG8gcGFzcyBpbnRvIEthaXNlciBJbXBhY3QgaXMgYmFubmVkXG4tIFNraWxscyBhcmUgYmFubmVkIHRvIHVzZSBpZiBhIGZsb3cgdGhhdCByZWR1Y2VzIGNvb2xkb3duIGlzIHBvcHBlZFxuIl0sIkZsb3cgUnVsZXMiOlsisIExpbWl0ZWRcblxuSG9tZWxlc3MgbWFuIEFsbG93ZWQ6XG4taG9tZWxlc3MgbWFuIGlzIHVuYmFubmVkIGJ1dCB1IGNhbiBvbmx5IHNob39tIGZyb20gbWlkZmllbClcblxuU3BhZGUgQWxsb3dlZCA6XG4tYnV0IHlvdSBjYW4gb25seSBzaG9vdCBmcm9tIG1pZGZpZWxkIiwiIyBNYXN0ZXIgZmxvd1xuXG5CdXR0ZXJmbHkgRGFuY2VyIEFsbG93ZWQgOlxuLUxhdmluaG8gZmxvdzsgY2FuIHVzZSAyIHBlciB0ZWFtIGJ1dCBvbmx5IDEgaWYgYnVubnkgXG4tZmxvdy9kZXN0cnVjdGl2ZSBpbXB1bHNlcyBpcyBwYWlyZWQgd2l0aCBpdC5cblxuR29kU3BlZWQgQmFubmVkIDpcbiIsIiMgR2VuZXJhdGlvbmFsIGZsb3cgXG5cbkF3YWtlbmVkIEdlbml1cyBBbGxvd2VkIDpcblxuRW1wZXJvciBBbGxvd2VkIDpcbi0gc2hvb3Rpbmcgd2l0aCBFbXBlcm9yIGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIGJhbm5lZC5cblxuQWNlIEVhdGVyIEFsbG93ZWQgOlxuXG5UYXJnZXQgTWFuIEFsbG93ZWQgOlxuLUJ1bm55IGZsb3codGFyZ2V0bWFuKTtPbmx5IDEgb2YgaXQgb24gc2FtZSB0ZWFtIG9mZmVuc2l2ZWx5KCBDTSBhbmQgR0sgY2FuIGR1cGUgaXQpLCBiYW5uZWQgd2l0aCBOZWwgUmVvICAsQmFubmVkIHRvIGJlIHBsYXllZCB3aXRoIGFub3RoZXIgZm9yd2FyZCB1c2luZyBkZW1vbiBraW5nLiBcbi1UYXJnZXQgbWFuIGZsb3cgdm9sbGV5IGJhbm5lZCBpbmJveCAobm9ybWFsIG0xIGFsbG93ZWQgaW5ib3ggKVxuIiwiIyBXb3JsZCBDbGFzcyBmbG93IFxuXG5EZW1vbiBraW5nIEFsbG93ZWQgOiBcbi0gT25seSAxIG9mIGl0IG9uIHNhbWUgdGVhbSAob2ZmZW5zaXZlbHksIENNIGFuZCBHSyBjYW5cbi0gZHVwZSBpdCBidXQganVzdCBvbmUgb2YgdGhlbSwgbm90IGJvdGgpLiBCYW5uZWQgdG8gYmUgcGxheWVkIHdpdGggYW5vdGhlciBmb3J3YXJkIHVzaW5nIHRhcmdldCBtYW4gb3IgcHJvZGlneS4gXG5cblNpbmd1bGFyaXR5IEFsbG93ZWQ6XG4tIG9ubHkgMSBwZXIgdGVhbS4geG5cbkNvbnRyYXJpYW4gQWxsb3dlZCA6XG5cbkRlc3RydWN0aXZlIGxtcHVsc2VzIChQcm9kaWd5KSBBbGxvd2VkIDpcbi0gU2hvdCBiYW5uZWQgaW5ib3guIE9ubHkgMSBvZiBpdCBvbiBzYW1lIHRlYW0gKG9mZmVuc2l2ZWx5LCBDTSBhbmQgR0sgY2FuIGR1cGUgaXQpXG4tIGhvb3Rpbmcgd2l0aCBkZXN0cnVjdGl2ZSBpbXB1bHNlcyBpbiBib3ggZHVyaW5nIGtpbmfigJlzIGF3YWtlbmluZyBpcyBiYW5uZWQuXG4iLCIjIE15dGhpYyBcblxuU25ha2UgQWxsb3dlZCA6XG5cbk1hZ2ljaWFuIEZsb3cgQWxsb3dlZCA6XG5cbk1hc3RlciBPZiBBbGwgVHJhZGVzIEFsbG93ZWQgOlxuYW4gYmUgdXNlZCBieSBhbnlvbmUgb24gdGVhbS4gKE1heCBsaW1pdDogbm9uKVxuXG5LaW5ncyBBdXRob3JpdHkgQWxsb3dlZCA6XG4tIHNob290aW5nIHdpdGggS2luZ3MgQXV0aG9yaXR5IGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIEJhbm5lZFxuIiwiIyBMZWdlbmRhcnkgXG5cbkNyb3cgQWxsb3dlZCA6XG4tIHNob290aW5nIHdpdGggY3JvdyBpbiBib3ggZHVyaW5nIGtpbmfigJlzIGF3YWtlbmluZyBpcyBiYW5uZWQuXG5cbkRyaWJibGVyIEFsbG93ZWQgOlxuLSBzaG9vdGluZyB3aXRoIGRyaWJibGVyIGluIGJveCBkdXJpbmcga2luZ+KAmXMgYXdha2VuaW5nIGlzIGJhbm5lZC5cblxuVHJhcCAoRmxvdykgQWxsb3dlZCA6XG5cbkRlbW9uIFdpbmdzIEFsbG93ZWQ6XG5cbldpbGQgQ2FyZCBBbGxvd2VkIDogXG5vbiIsIiMgRXBpYyBcblxuR2VuaXVzIEFsbG93ZWQgOiBcblxuTW9uc3RlciBBbGxvd2VkIDogXG5cbkljZSBBbGxvd2VkIDogXG5cblN0ZWFsdGggQWxsb3dlZCA6XG4iLCIjIFJhcmVcblxuTGlnaHRuaW5nIEFsbG93ZWQgOlxuXG5QdXp6bGUgQWxsb3dlZCA6XG5cbkJ1ZGRoYXMgQmxlc3NpbmcgQWxsb3dlZCA6XG4iXSwiR2xvYmFsIE5ld3MgJiBBcmNoaXZlIjpbXX0sImFjY3MiOlt7InVzZXIiOiJyaXBfY2VuayIsInBhc3MiOiIzMjIwMjgwMDQ2NjUwMTAwIiwicm9sZSI6Ik9XTkVSIiwia2V5MmZhIjoiMjIwOTEwIiwiYXZhdGFyIjoiaHR0cHM6Ly9pLnBpbmltZy5jb20vNzM2eC9hYS80Yy80NC9hYTRjNDRiYzEwZTdjMDU1MTgxMGQyMGMxMjJjNThiNy5qcGcifSx7InVzZXIiOiJ4bTIwMDJhbiIsInBhc3MiOiIzMjIwMjgwMCIsInJvbGUiOiJNRU1CRVIiLCJrZXkyZmEiOm51bGwsIm5pY2siOiJ4bTIwMDJhbiIsImF2YXRhciI6IiIsInBlcm0iOiJub25lIn1dLCJyYW5rcyI6W3sibmFtZSI6IkFETUlOIiwicGVybSI6ImFsbCJ9LHsibmFtZSI6Ik1FTUJFUiIsInBlcm0iOiJub25lIn0seyJuYW1lIjoiRmxvdyBydWxlcyBlZGl0b3IiLCJwZXJtIjoiRmxvdyBSdWxlcyJ9LHsibmFtZSI6IlN0eWxlIHJ1bGVzIGVkaXRvciIsInBlcm0iOiJTdHlsZSBSdWxlcyJ9LHsibmFtZSI6IlJhbmtlZCBydWxlcyBlZGl0b3IiLCJwZXJtIjoiUmFua2VkIFJ1bGVzIn0seyJuYW1lIjoiRmxvdyBFZGl0b3IiLCJwZXJtIjoiRmxvdyBSdWxlcyJ9XX0=";

        let joudDB = {};
        let activeCategory = "";
        let currentUser = { user: "rip_cenk", role: "OWNER" }; // افتراضياً أنت الأدمن

        // وظيفة الحقن التلقائي
        function autoInjectData() {
            try {
                const decoded = JSON.parse(decodeURIComponent(escape(atob(MASTER_BACKUP_KEY))));
                joudDB = decoded.db;
                localStorage.setItem('joud_core', JSON.stringify(joudDB));
                console.log("System: Data Injected Successfully");
            } catch(e) {
                console.error("Injection Error:", e);
            }
        }

        // تشغيل الحقن عند بدء البرنامج
        window.onload = () => {
            if(!localStorage.getItem('joud_core')) {
                autoInjectData();
            } else {
                joudDB = JSON.parse(localStorage.getItem('joud_core'));
            }
            updateProfileUI();
        };

        function openPage(cat) {
            activeCategory = cat;
            document.getElementById('main-menu').style.display = 'none';
            document.getElementById('page-view').style.display = 'block';
            document.getElementById('back-nav-btn').style.visibility = 'visible';
            document.getElementById('page-title').innerText = cat;
            document.getElementById('admin-panel').style.display = (currentUser.role === "OWNER") ? 'block' : 'none';
            renderEntries();
        }

        function renderEntries() {
            const container = document.getElementById('data-list');
            const searchVal = document.getElementById('rule-search').value.toLowerCase();
            const data = joudDB[activeCategory] || [];
            container.innerHTML = "";
            
            data.forEach((item, index) => {
                if(searchVal && !item.toLowerCase().includes(searchVal)) return;
                let lines = item.split('\n');
                let title = lines[0].startsWith('#') ? lines[0].replace('#', '') : 'Rule #' + (index + 1);

                const entry = document.createElement('div');
                entry.className = 'entry-item';
                entry.innerHTML = `
                    <div class="entry-header" onclick="toggleContent(${index})">
                        <span>${title}</span>
                        <span>▼</span>
                    </div>
                    <div class="entry-content" id="content-${index}">
                        <div>${lines.map(l => formatLine(l)).join('')}</div>
                    </div>
                `;
                container.appendChild(entry);
            });
        }

        function formatLine(line) {
            if (!line.trim()) return '<br>';
            const isImage = line.match(/\.(jpeg|jpg|gif|png|webp)/i) || line.includes('ibb.co') || line.includes('discordapp.com');
            if(isImage) return `<img src="${line.trim()}" class="rule-img">`;
            
            let res = line.replace(/\((.*?)\)/g, '<span class="bracket-text">($1)</span>')
                          .replace(/\bBanned\b/gi, '<span class="word-banned">Banned</span>')
                          .replace(/\bAllowed\b/gi, '<span class="word-allowed">Allowed</span>');
            return `<div>${res}</div>`;
        }

        function toggleContent(id) {
            const c = document.getElementById('content-' + id);
            c.style.display = (c.style.display === 'block') ? 'none' : 'block';
        }

        function closePage() {
            document.getElementById('main-menu').style.display = 'flex';
            document.getElementById('page-view').style.display = 'none';
            document.getElementById('back-nav-btn').style.visibility = 'hidden';
        }

        function updateProfileUI() {
            document.getElementById('display-name').innerText = currentUser.user + " (" + currentUser.role + ")";
        }

        // اختصارات الكيبورد لك يا جود
        window.addEventListener('keydown', e => {
            if(e.key === "F7") { // عرض الكود الحالي أو استعادة
                e.preventDefault();
                document.getElementById('backup-modal').style.display = 'flex';
                document.getElementById('backup-key-box').value = MASTER_BACKUP_KEY;
            }
            if(e.key === "F8") { // مزامنة إجبارية
                e.preventDefault();
                autoInjectData();
                location.reload();
            }
        });

        function restoreBackup() {
            const key = document.getElementById('backup-key-box').value;
            if(confirm("Force sync database?")) {
                localStorage.setItem('joud_core', JSON.stringify(JSON.parse(atob(key)).db));
                location.reload();
            }
        }
    </script>
</body>
</html>
