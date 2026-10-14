<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Leitor e Somador de XML (CT-e e NF-e)</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f7f6; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 1100px; margin: 0 auto; background: #fff; padding: 30px; border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
        h1 { text-align: center; color: #2c3e50; margin-bottom: 30px; }
        #drop-zone { border: 3px dashed #3498db; border-radius: 8px; padding: 50px 20px; text-align: center; background-color: #f8fafc; cursor: pointer; transition: all 0.3s ease; }
        #drop-zone.hover { background-color: #e2e8f0; border-color: #2ecc71; transform: scale(0.99); }
        #drop-zone p { margin: 0; font-size: 18px; color: #475569; }
        #file-input { display: none; }
        .cards-summary { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 20px; margin-top: 30px; }
        .card { background: #fff; padding: 20px; border-radius: 6px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); border-top: 4px solid #3498db; text-align: center; }
        .card.frete { border-top-color: #2ecc71; }
        .card.peso { border-top-color: #e67e22; }
        .card.mercadoria { border-top-color: #9b59b6; }
        .card h3 { margin: 0 0 10px 0; color: #7f8c8d; font-size: 14px; text-transform: uppercase; }
        .card .value { font-size: 24px; font-weight: bold; color: #2c3e50; }
        .results-table { width: 100%; border-collapse: collapse; margin-top: 30px; font-size: 14px; }
        .results-table th, .results-table td { padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }
        .results-table th { background-color: #f8f9fa; color: #2c3e50; }
        .results-table td.num, .results-table th.num { text-align: right; }
        .counter { margin-top: 15px; font-size: 14px; color: #475569; font-weight: 500; }
        .badge { padding: 4px 8px; border-radius: 4px; font-size: 11px; font-weight: bold; color: #fff; }
        .badge-cte { background-color: #2ecc71; }
        .badge-nfe { background-color: #3498db; }
        .btn { margin-top: 15px; padding: 8px 16px; border: 0; border-radius: 4px; background: #3498db; color: #fff; cursor: pointer; font-size: 14px; }
        .btn:hover { background: #2980b9; }
    </style>
</head>
<body>

<div class="container">
    <h1>Somador de XML (CT-e e NF-e)</h1>

    <div id="drop-zone">
        <p>🚀 <strong>Arraste e solte seus XMLs (CT-e ou NF-e) aqui</strong> ou clique para selecionar</p>
        <span style="font-size: 13px; color: #94a3b8; display: block; margin-top: 10px;">Arquivos inválidos, duplicados ou de outros tipos serão ignorados automaticamente</span>
        <input type="file" id="file-input" multiple accept=".xml">
    </div>

    <div class="counter" id="file-counter">Nenhum documento processado.</div>

    <div class="cards-summary">
        <div class="card frete">
            <h3>Total Frete</h3>
            <div class="value" id="total-frete">R$ 0,00</div>
        </div>
        <div class="card peso">
            <h3>Total Peso Bruto</h3>
            <div class="value" id="total-peso">0,00 kg</div>
        </div>
        <div class="card mercadoria">
            <h3>Total Mercadoria / Notas</h3>
            <div class="value" id="total-mercadoria">R$ 0,00</div>
        </div>
    </div>

    <button class="btn" id="btn-csv" style="display:none;">⬇ Exportar CSV</button>

    <table class="results-table" id="results-table" style="display: none;">
        <thead>
            <tr>
                <th>Tipo</th>
                <th>Número</th>
                <th>CNPJ/CPF Destino</th>
                <th>Destinatário</th>
                <th class="num">Valor Frete</th>
                <th class="num">Peso Bruto</th>
                <th class="num">Valor Mercadoria</th>
            </tr>
        </thead>
        <tbody id="table-body"></tbody>
    </table>
</div>

<script>
    const dropZone = document.getElementById('drop-zone');
    const fileInput = document.getElementById('file-input');
    const fileCounter = document.getElementById('file-counter');
    const totalFreteEl = document.getElementById('total-frete');
    const totalPesoEl = document.getElementById('total-peso');
    const totalMercadoriaEl = document.getElementById('total-mercadoria');
    const resultsTable = document.getElementById('results-table');
    const tableBody = document.getElementById('table-body');
    const btnCsv = document.getElementById('btn-csv');

    let registros = []; // documentos válidos já lidos

    dropZone.addEventListener('click', () => fileInput.click());
    fileInput.addEventListener('change', (e) => { handleFiles(e.target.files); fileInput.value = ''; });

    window.addEventListener("dragover", (e) => e.preventDefault(), false);
    window.addEventListener("drop", (e) => e.preventDefault(), false);

    dropZone.addEventListener('dragenter', (e) => { e.preventDefault(); dropZone.classList.add('hover'); });
    dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.classList.add('hover'); });
    dropZone.addEventListener('dragleave', () => dropZone.classList.remove('hover'));
    dropZone.addEventListener('drop', (e) => {
        e.preventDefault();
        dropZone.classList.remove('hover');
        const files = e.dataTransfer.items
            ? Array.from(e.dataTransfer.items).filter(i => i.kind === 'file').map(i => i.getAsFile())
            : e.dataTransfer.files;
        handleFiles(files);
    });

    const formatCurrency = (v) => new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(v);
    const formatNumber = (v) => new Intl.NumberFormat('pt-BR', { minimumFractionDigits: 2, maximumFractionDigits: 3 }).format(v);

    function formatDoc(d) {
        if (!d) return 'N/A';
        if (d.length === 14) return d.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, '$1.$2.$3/$4-$5');
        if (d.length === 11) return d.replace(/^(\d{3})(\d{3})(\d{3})(\d{2})$/, '$1.$2.$3-$4');
        return d;
    }

    // --- Auxiliares: leitura precisa (filhos diretos), sem pegar o primeiro item que aparecer ---
    const first = (root, tag) => root ? root.getElementsByTagName(tag)[0] : null;
    function child(el, tag) {
        if (!el) return null;
        for (const c of el.children) if (c.localName === tag) return c;
        return null;
    }
    const num = (el) => {
        const v = parseFloat((el?.textContent || '0').trim());
        return isNaN(v) ? 0 : v;
    };
    const txt = (el) => (el?.textContent || '').trim();

    function parseDocumento(xmlDoc) {
        // ---------- NF-e ----------
        const infNFe = first(xmlDoc, "infNFe");
        if (infNFe) {
            const chave = (infNFe.getAttribute("Id") || '').replace(/^NFe/, '');
            const ide = child(infNFe, "ide");
            const dest = child(infNFe, "dest");
            const total = child(infNFe, "total");
            const icmsTot = child(total, "ICMSTot");

            // Valor total da nota (vNF). Se não existir, usa vProd do TOTAL (nunca o do item).
            let valorNota = num(child(icmsTot, "vNF"));
            if (!valorNota) valorNota = num(child(icmsTot, "vProd"));
            if (!valorNota) { // último recurso: soma dos itens
                for (const det of infNFe.children) {
                    if (det.localName === "det") valorNota += num(child(child(det, "prod"), "vProd"));
                }
            }

            const vFrete = num(child(icmsTot, "vFrete"));

            // Peso bruto: soma de TODOS os volumes (<transp><vol>)
            let pesoBruto = 0;
            const transp = child(infNFe, "transp");
            if (transp) {
                for (const vol of transp.children) {
                    if (vol.localName === "vol") pesoBruto += num(child(vol, "pesoB"));
                }
            }

            const docDest = txt(child(dest, "CNPJ")) || txt(child(dest, "CPF")) || txt(child(dest, "idEstrangeiro"));

            return {
                tipo: "NF-e",
                chave,
                numero: txt(child(ide, "nNF")) || "N/A",
                docDestino: docDest,
                destinatario: txt(child(dest, "xNome")),
                frete: vFrete,
                peso: pesoBruto,
                mercadoria: valorNota
            };
        }

        // ---------- CT-e ----------
        const infCte = first(xmlDoc, "infCte");
        if (infCte) {
            const chave = (infCte.getAttribute("Id") || '').replace(/^CTe/, '');
            const ide = child(infCte, "ide");
            const vPrest = child(infCte, "vPrest");
            const infCTeNorm = child(infCte, "infCTeNorm");
            const infCarga = child(infCTeNorm, "infCarga");
            const dest = child(infCte, "dest");

            // Peso: prefere "PESO BRUTO"; senão, primeiro infQ
            let pesoBruto = 0, achou = false, primeiro = null;
            if (infCarga) {
                for (const q of infCarga.children) {
                    if (q.localName !== "infQ") continue;
                    const v = num(child(q, "qCarga"));
                    if (primeiro === null) primeiro = v;
                    const tp = txt(child(q, "tpMed")).toUpperCase();
                    if (tp.includes("PESO BRUTO") || tp.includes("PESO_BRUTO")) { pesoBruto = v; achou = true; break; }
                }
                if (!achou && primeiro !== null) pesoBruto = primeiro;
            }

            const docDest = txt(child(dest, "CNPJ")) || txt(child(dest, "CPF"));

            return {
                tipo: "CT-e",
                chave,
                numero: txt(child(ide, "nCT")) || "N/A",
                docDestino: docDest,
                destinatario: txt(child(dest, "xNome")),
                frete: num(child(vPrest, "vTPrest")),
                peso: pesoBruto,
                mercadoria: num(child(infCarga, "vCarga"))
            };
        }

        return null;
    }

    function readFile(file) {
        return new Promise((resolve) => {
            const reader = new FileReader();
            reader.onload = (e) => {
                try {
                    const xmlDoc = new DOMParser().parseFromString(e.target.result, "text/xml");
                    if (xmlDoc.getElementsByTagName("parsererror").length > 0) return resolve(null);
                    resolve(parseDocumento(xmlDoc));
                } catch (err) { resolve(null); }
            };
            reader.onerror = () => resolve(null);
            reader.readAsText(file);
        });
    }

    async function handleFiles(files) {
        const xmlFiles = Array.from(files).filter(f => f && f.name.toLowerCase().endsWith('.xml'));
        if (xmlFiles.length === 0) return;

        const resultados = await Promise.all(xmlFiles.map(readFile));
        const chavesVistas = new Set(registros.map(r => r.chave).filter(Boolean));
        let duplicados = 0;

        resultados.forEach(r => {
            if (!r) return;
            if (r.chave && chavesVistas.has(r.chave)) { duplicados++; return; }
            if (r.chave) chavesVistas.add(r.chave);
            registros.push(r);
        });

        render(duplicados);
    }

    function render(duplicados = 0) {
        let totalFrete = 0, totalPeso = 0, totalMercadoria = 0;
        tableBody.innerHTML = '';

        registros.forEach(r => {
            totalFrete += r.frete;
            totalPeso += r.peso;
            totalMercadoria += r.mercadoria;

            const badgeClass = r.tipo === "CT-e" ? "badge-cte" : "badge-nfe";
            const tr = document.createElement('tr');
            tr.innerHTML = `
                <td><span class="badge ${badgeClass}">${r.tipo}</span></td>
                <td><strong>${r.numero}</strong></td>
                <td>${formatDoc(r.docDestino)}</td>
                <td>${r.destinatario || '-'}</td>
                <td class="num">${formatCurrency(r.frete)}</td>
                <td class="num">${formatNumber(r.peso)} kg</td>
                <td class="num">${formatCurrency(r.mercadoria)}</td>`;
            tableBody.appendChild(tr);
        });

        if (registros.length > 0) {
            resultsTable.style.display = 'table';
            btnCsv.style.display = 'inline-block';
            let msg = `✅ <strong style="color: #2ecc71;">${registros.length}</strong> documento(s) válido(s) calculado(s) com sucesso.`;
            if (duplicados > 0) msg += ` (${duplicados} duplicado(s) ignorado(s))`;
            fileCounter.innerHTML = msg;
        } else {
            resultsTable.style.display = 'none';
            btnCsv.style.display = 'none';
            fileCounter.innerHTML = `⚠️ Nenhum documento do tipo CT-e ou NF-e válido foi localizado.`;
        }

        totalFreteEl.textContent = formatCurrency(totalFrete);
        totalPesoEl.textContent = `${formatNumber(totalPeso)} kg`;
        totalMercadoriaEl.textContent = formatCurrency(totalMercadoria);
    }

    btnCsv.addEventListener('click', () => {
        const linhas = [["Tipo","Numero","CNPJ/CPF Destino","Destinatario","Valor Frete","Peso Bruto (kg)","Valor Mercadoria"]];
        registros.forEach(r => linhas.push([
            r.tipo, r.numero, r.docDestino, `"${(r.destinatario || '').replace(/"/g, '""')}"`,
            r.frete.toFixed(2).replace('.', ','), r.peso.toFixed(3).replace('.', ','), r.mercadoria.toFixed(2).replace('.', ',')
        ]));
        const csv = "\uFEFF" + linhas.map(l => l.join(';')).join('\n');
        const a = document.createElement('a');
        a.href = URL.createObjectURL(new Blob([csv], { type: 'text/csv;charset=utf-8;' }));
        a.download = 'resumo_xml.csv';
        a.click();
    });
</script>

</body>
</html>
