<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Katalog Asheeqa Printing</title>
    <style>
        :root { --wa: #25D366; --promo: #d63031; --dark: #2d3436; --orange: #e67e22; }
        body { font-family: 'Segoe UI', sans-serif; background: #f0f2f5; margin: 0; padding-bottom: 80px; }
        
        /* Navigation */
        .nav-tabs { display: flex; justify-content: center; background: white; position: sticky; top: 0; z-index: 100; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .tab { padding: 15px 20px; cursor: pointer; font-weight: bold; color: #636e72; border-bottom: 3px solid transparent; }
        .tab.active { color: var(--promo); border-bottom: 3px solid var(--promo); }

        /* Admin Box */
        .admin-box { background: white; padding: 20px; border-radius: 12px; max-width: 500px; margin: 20px auto; display: none; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        input, select, button { width: 100%; padding: 12px; margin: 5px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        .btn-save { background: #0984e3; color: white; border: none; font-weight: bold; cursor: pointer; }

        /* Grid Katalog */
        .container { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; padding: 20px; max-width: 1200px; margin: 0 auto; }
        .card { background: white; border-radius: 15px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.05); position: relative; transition: 0.3s; }
        .card img { width: 100%; height: 250px; object-fit: cover; }
        .promo-tag { position: absolute; top: 10px; left: 10px; background: var(--promo); color: white; padding: 4px 10px; border-radius: 5px; font-weight: bold; font-size: 12px; }
        .jenis-tag { position: absolute; top: 10px; right: 10px; background: rgba(255,255,255,0.8); padding: 4px 10px; border-radius: 5px; font-size: 11px; font-weight: bold; }
        
        .info { padding: 15px; text-align: center; }
        .p-old { text-decoration: line-through; color: #b2bec3; font-size: 0.9em; }
        .p-new { color: var(--promo); font-size: 1.2em; font-weight: bold; display: block; margin-bottom: 10px; }
        .btn-cart { background: var(--dark); color: white; border: none; padding: 10px; border-radius: 8px; cursor: pointer; width: 100%; font-weight: bold; }

        /* Floating Cart Icon */
        .cart-float { position: fixed; bottom: 20px; right: 20px; background: var(--orange); color: white; padding: 15px 25px; border-radius: 50px; cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.2); z-index: 1000; }

        /* Modal Keranjang */
        #modal-cart { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 2000; justify-content: center; align-items: center; }
        .modal-body { background: white; padding: 20px; border-radius: 15px; width: 90%; max-width: 400px; max-height: 70vh; overflow-y: auto; }
    </style>
</head>
<body>

    <h1 style="text-align: center; padding: 20px 0; margin: 0; background: white;">Asheeqa Printing</h1>

    <div class="nav-tabs">
        <div class="tab active" onclick="filter('PROMO', this)">🔥 PROMO</div>
        <div class="tab" onclick="filter('AD', this)">UNDANGAN AD</div>
        <div class="tab" onclick="filter('CG', this)">UNDANGAN CG</div>
    </div>

    <div id="admin-box" class="admin-box">
        <h3>Tambah Produk Baru</h3>
        <input type="text" id="nama" placeholder="Nama Undangan">
        <select id="kelompok"><option value="AD">Kelompok AD</option><option value="CG">Kelompok CG</option></select>
        <select id="jenis"><option value="Hotprint">Hotprint</option><option value="Tidak Hotprint">Tidak Hotprint</option></select>
        <input type="number" id="hargaA" placeholder="Harga Asli">
        <input type="number" id="hargaP" placeholder="Harga Promo (Kosongkan jika tidak ada)">
        <input type="file" id="foto" accept="image/*">
        <button onclick="upload()" id="btnSave" class="btn-save">Simpan ke Katalog</button>
    </div>

    <div class="container" id="container">Memuat data...</div>

    <div class="cart-float" onclick="toggleCart()">🛒 Keranjang (<span id="cart-count">0</span>)</div>

    <div id="modal-cart">
        <div class="modal-body">
            <h3>Daftar Pesanan</h3>
            <div id="cart-list"></div>
            <hr>
            <div style="display:flex; justify-content:space-between; font-weight:bold;">
                <span>Total:</span><span id="cart-total">Rp 0</span>
            </div>
            <button onclick="checkoutWA()" style="background:var(--wa); color:white; margin-top:15px;">Kirim ke WhatsApp</button>
            <button onclick="toggleCart()" style="background:#eee; color:black; margin-top:5px;">Tutup</button>
        </div>
    </div>

    <script>
        const scriptURL = 'https://script.google.com/macros/s/AKfycbyybtx0ZkCPTRcI83feg66Qffmdxfbsf_6VvKQ0THMZvsZitPMTl7huDsFiB-KDgtJcrA/exec';
        const noWA = '089699910769'; // Ganti nomor Anda
        let database = [];
        let cart = [];

        const isAdmin = new URLSearchParams(window.location.search).get('mode') === 'admin';
        if (isAdmin) document.getElementById('admin-box').style.display = 'block';

        async function muatData() {
            const r = await fetch(scriptURL + '?action=read');
            database = await r.json();
            filter('PROMO', document.querySelector('.tab'));
        }

        function filter(kat, el) {
            document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
            el.classList.add('active');
            const cont = document.getElementById('container');
            cont.innerHTML = '';

            let filtered = kat === 'PROMO' ? database.filter(i => i.hargaPromo !== "") : database.filter(i => i.kelompok === kat);

            filtered.forEach(i => {
                const isP = i.hargaPromo !== "";
                const hFinal = isP ? i.hargaPromo : i.hargaAsli;
                const hHtml = isP ? `<span class="p-old">Rp ${parseInt(i.hargaAsli).toLocaleString()}</span><span class="p-new">Rp ${parseInt(i.hargaPromo).toLocaleString()}</span>` : `<span class="p-new">Rp ${parseInt(i.hargaAsli).toLocaleString()}</span>`;
                
                cont.innerHTML += `
                    <div class="card">
                        ${isP ? '<div class="promo-tag">PROMO</div>' : ''}
                        <div class="jenis-tag">${i.jenis}</div>
                        <img src="${i.url}">
                        <div class="info">
                            <strong>${i.nama}</strong>
                            <div style="margin:10px 0">${hHtml}</div>
                            ${isAdmin ? `<button onclick="edit('${i.id}','${i.nama}','${i.hargaAsli}','${i.hargaPromo}','${i.jenis}')" style="background:#f1c40f; margin-bottom:5px">✏️ Edit</button>` : ''}
                            <button class="btn-cart" onclick="addToCart('${i.nama}', ${hFinal})">+ Keranjang</button>
                        </div>
                    </div>`;
            });
        }

        function addToCart(nama, harga) {
            cart.push({ nama, harga });
            document.getElementById('cart-count').innerText = cart.length;
            alert(nama + " masuk keranjang!");
        }

        function toggleCart() {
            const m = document.getElementById('modal-cart');
            m.style.display = m.style.display === 'flex' ? 'none' : 'flex';
            renderCart();
        }

        function renderCart() {
            const list = document.getElementById('cart-list');
            let total = 0; list.innerHTML = '';
            cart.forEach((item, idx) => {
                total += item.harga;
                list.innerHTML += `<div style="display:flex; justify-content:space-between; margin-bottom:5px;">
                    <span>${item.nama}</span>
                    <span>Rp ${item.harga.toLocaleString()} <b style="color:red; cursor:pointer" onclick="removeItem(${idx})">x</b></span>
                </div>`;
            });
            document.getElementById('cart-total').innerText = "Rp " + total.toLocaleString();
        }

        function removeItem(idx) { cart.splice(idx, 1); document.getElementById('cart-count').innerText = cart.length; renderCart(); }

        function checkoutWA() {
            if(cart.length === 0) return;
            let txt = "Halo Asheeqa Printing, saya ingin pesan:%0A";
            let total = 0;
            cart.forEach((i, idx) => { txt += `${idx+1}. ${i.nama} (Rp ${i.harga.toLocaleString()})%0A`; total += i.harga; });
            txt += `%0A*Total: Rp ${total.toLocaleString()}*`;
            window.open(`https://api.whatsapp.com/send?phone=${noWA}&text=${txt}`, '_blank');
        }

        function upload() {
            const f = document.getElementById('foto').files[0];
            if(!f) return;
            document.getElementById('btnSave').innerText = "Proses...";
            const r = new FileReader(); r.readAsDataURL(f);
            r.onload = function() {
                const b64 = r.result.split(',')[1];
                const p = new URLSearchParams({ filename: f.name, nama: document.getElementById('nama').value, kelompok: document.getElementById('kelompok').value, jenis: document.getElementById('jenis').value, hargaAsli: document.getElementById('hargaA').value, hargaPromo: document.getElementById('hargaP').value });
                fetch(`${scriptURL}?${p}`, { method: 'POST', body: b64 }).then(() => location.reload());
            };
        }

        function edit(id, n, hA, hP, j) {
            const nB = prompt("Nama:", n); const hAB = prompt("Harga Asli:", hA); const hPB = prompt("Harga Promo:", hP); const jB = prompt("Jenis:", j);
            const p = new URLSearchParams({ action: 'update', rowId: id, nama: nB, hargaAsli: hAB, hargaPromo: hPB, jenis: jB });
            fetch(`${scriptURL}?${p}`, { method: 'POST' }).then(() => location.reload());
        }

        muatData();
    </script>
</body>
</html>
