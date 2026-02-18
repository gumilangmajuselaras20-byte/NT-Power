<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kalkulator Tagihan PLN Industri</title>
    <style>
        :root {
            --primary: #2c3e50;
            --success: #2ecc71;
            --danger: #e74c3c;
            --bg: #f4f6f8;
        }

        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            background: var(--bg); 
            padding: 20px;
            color: #333;
            line-height: 1.6;
        }

        .container { max-width: 800px; margin: 0 auto; }
        
        h2 { text-align: center; color: var(--primary); margin-bottom: 30px; }

        .grid { 
            display: grid; 
            grid-template-columns: 1fr 1fr; 
            gap: 20px; 
        }

        @media (max-width: 600px) { .grid { grid-template-columns: 1fr; } }

        .box { 
            background: #fff; 
            padding: 25px; 
            border-radius: 12px; 
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        h3 { margin-top: 0; border-bottom: 2px solid #eee; padding-bottom: 10px; font-size: 1.1rem; }

        .input-group { margin-bottom: 15px; }

        label { display: block; margin-bottom: 5px; font-weight: 600; font-size: 0.9rem; }

        input { 
            width: 100%; 
            padding: 10px; 
            border: 1px solid #ddd; 
            border-radius: 6px; 
            box-sizing: border-box;
            transition: border 0.3s;
        }

        input:focus { border-color: var(--success); outline: none; }

        button { 
            margin-top: 20px; 
            padding: 15px; 
            width: 100%; 
            background: var(--success); 
            border: none; 
            color: #fff; 
            font-size: 18px; 
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.3s;
        }

        button:hover { background: #27ae60; }

        .result-box { 
            margin-top: 25px;
            background: #fff;
            border-left: 5px solid var(--success);
        }

        .result-item { display: flex; justify-content: space-between; margin-bottom: 8px; }
        .total-row { border-top: 2px solid #eee; padding-top: 10px; margin-top: 10px; }
        .final-price { color: var(--danger); font-size: 1.4rem; font-weight: 800; }
    </style>
</head>

<body>

<div class="container">
    <h2>⚡ Kalkulator PLN Industri</h2>

    <div class="grid">
        <div class="box">
            <h3>📊 Data Pemakaian</h3>
            <div class="input-group">
                <label>Daya Kontrak (kVA)</label>
                <input type="number" id="kva" placeholder="Contoh: 200">
            </div>
            <div class="input-group">
                <label>kWh LWBP (Luar Waktu Beban Puncak)</label>
                <input type="number" id="kwh_lwbp" placeholder="0">
            </div>
            <div class="input-group">
                <label>kWh WBP (Waktu Beban Puncak)</label>
                <input type="number" id="kwh_wbp" placeholder="0">
            </div>
            <div class="input-group">
                <label>Total kVARh</label>
                <input type="number" id="kvarh" placeholder="0">
            </div>
        </div>

        <div class="box">
            <h3>💰 Parameter Tarif (Rp)</h3>
            <div class="input-group">
                <label>Tarif LWBP /kWh</label>
                <input type="number" id="tarif_lwbp" value="1035">
            </div>
            <div class="input-group">
                <label>Tarif WBP /kWh</label>
                <input type="number" id="tarif_wbp" value="1553">
            </div>
            <div class="input-group">
                <label>Tarif Beban /kVA</label>
                <input type="number" id="tarif_beban" value="45000">
            </div>
            <div class="input-group">
                <label>Tarif Denda kVARh</label>
                <input type="number" id="tarif_kvarh" value="1100">
            </div>
            <div class="input-group">
                <label>PPJ (%)</label>
                <input type="number" id="ppj" value="3">
            </div>
        </div>
    </div>

    <button onclick="hitung()">Hitung Rincian Tagihan</button>

    <div class="box result-box" id="resultArea" style="display:none;">
        <h3>📋 Rincian Estimasi Biaya</h3>
        <div id="detail_output"></div>
    </div>
</div>

<script>
function hitung() {
    // Ambil Value
    const kva = parseFloat(document.getElementById("kva").value) || 0;
    const kwh_lwbp = parseFloat(document.getElementById("kwh_lwbp").value) || 0;
    const kwh_wbp = parseFloat(document.getElementById("kwh_wbp").value) || 0;
    const kvarh = parseFloat(document.getElementById("kvarh").value) || 0;

    const t_lwbp = parseFloat(document.getElementById("tarif_lwbp").value) || 0;
    const t_wbp = parseFloat(document.getElementById("tarif_wbp").value) || 0;
    const t_beban = parseFloat(document.getElementById("tarif_beban").value) || 0;
    const t_kvarh = parseFloat(document.getElementById("tarif_kvarh").value) || 0;
    const p_ppj = parseFloat(document.getElementById("ppj").value) / 100 || 0;

    // Kalkulasi
    const biaya_lwbp = kwh_lwbp * t_lwbp;
    const biaya_wbp = kwh_wbp * t_wbp;
    const biaya_beban = kva * t_beban;

    // Rumus kVARh (Batas 0.85 cos phi)
    const kwh_total = kwh_lwbp + kwh_wbp;
    const kvarh_allow = kwh_total * 0.6197; // Hasil dari Tan(Acos(0.85))
    const kvarh_excess = Math.max(0, kvarh - kvarh_allow);
    const denda_kvarh = kvarh_excess * t_kvarh;

    const subtotal = biaya_lwbp + biaya_wbp + biaya_beban + denda_kvarh;
    const pajak = subtotal * p_ppj;
    const total = subtotal + pajak;

    // Tampilkan Hasil
    const fmt = (num) => "Rp " + Math.round(num).toLocaleString("id-ID");
    
    document.getElementById("resultArea").style.display = "block";
    document.getElementById("detail_output").innerHTML = `
        <div class="result-item"><span>Biaya Beban (Abodemen):</span> <span>${fmt(biaya_beban)}</span></div>
        <div class="result-item"><span>Pemakaian LWBP:</span> <span>${fmt(biaya_lwbp)}</span></div>
        <div class="result-item"><span>Pemakaian WBP:</span> <span>${fmt(biaya_wbp)}</span></div>
        <div class="result-item"><span>Denda kVARh:</span> <span>${fmt(denda_kvarh)}</span></div>
        <div class="result-item"><span>PPJ:</span> <span>${fmt(pajak)}</span></div>
        <div class="result-item total-row">
            <strong>TOTAL TAGIHAN:</strong> 
            <span class="final-price">${fmt(total)}</span>
        </div>
    `;
}
</script>

</body>
</html>
