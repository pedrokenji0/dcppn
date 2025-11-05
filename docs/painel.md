# 🧾 Painel de Atualização – DCPPN

> Ferramenta interna para criação automática de **Atos DCPPN** diretamente no repositório GitHub.

---

<form id="novoAtoForm" style="display:flex;flex-direction:column;gap:10px;max-width:700px;">
  <label><b>Número do Ato</b></label>
  <input type="text" id="numero" placeholder="Ex: 149" required>

  <label><b>Título</b></label>
  <input type="text" id="titulo" placeholder="Ex: Modificação no Classificador de Elemento-item" required>

  <label><b>Responsáveis</b></label>
  <input type="text" id="responsavel" placeholder="Ex: Gabriel / Carolina" required>

  <label><b>Órgão demandante</b></label>
  <input type="text" id="orgao" placeholder="Ex: PMMG" required>

  <label><b>Data</b></label>
  <input type="text" id="data" placeholder="Ex: 09/04/2025" required>

  <label><b>Tipo de Modificação</b></label>
  <select id="tipo">
    <option value="BLOQUEIO">Bloqueio</option>
    <option value="RECLASSIFICACAO">Reclassificação</option>
    <option value="ADICAO">Adição</option>
    <option value="CRIACAO">Criação</option>
  </select>

  <label><b>Elemento-item proposto</b></label>
  <input type="text" id="elemento" placeholder="Ex: 3002 – Artigos para Esporte">

  <label><b>Elemento-item a ser substituído (caso aplicável)</b></label>
  <input type="text" id="substituido" placeholder="Ex: 3001 – Materiais esportivos">

  <label><b>Contexto da demanda</b></label>
  <textarea id="contexto" rows="6" placeholder="Descreva o contexto da demanda"></textarea>

  <label><b>Solução encontrada</b></label>
  <textarea id="solucao" rows="6" placeholder="Descreva a solução encontrada"></textarea>

  <label><b>Anexar PDF (opcional)</b></label>
  <input type="file" id="pdfFile" accept="application/pdf">
  <iframe id="pdfPreview" style="width:100%;height:400px;border:1px solid #ccc;display:none;"></iframe>

  <button type="submit" style="padding:10px;background-color:#1976D2;color:white;border:none;border-radius:5px;">Gerar e Salvar Ato</button>
</form>

<script>
const repo = "pedrokenji0/dcppn";
const branch = "main";

const pdfInput = document.getElementById("pdfFile");
const pdfPreview = document.getElementById("pdfPreview");

pdfInput.addEventListener("change", () => {
  const file = pdfInput.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      pdfPreview.src = e.target.result;
      pdfPreview.style.display = "block";
    };
    reader.readAsDataURL(file);
  } else {
    pdfPreview.style.display = "none";
  }
});

document.getElementById("novoAtoForm").addEventListener("submit", async (e) => {
  e.preventDefault();

  const n = document.getElementById("numero").value.trim();
  const titulo = document.getElementById("titulo").value.trim();
  const responsavel = document.getElementById("responsavel").value.trim();
  const orgao = document.getElementById("orgao").value.trim();
  const data = document.getElementById("data").value.trim();
  const tipo = document.getElementById("tipo").value.trim();
  const elemento = document.getElementById("elemento").value.trim();
  const substituido = document.getElementById("substituido").value.trim();
  const contexto = document.getElementById("contexto").value.trim();
  const solucao = document.getElementById("solucao").value.trim();

  const markdown = `
# Ato ${n}

## ${titulo}

**RESPONSÁVEL:** ${responsavel}  
**ÓRGÃO DEMANDANTE:** ${orgao}  
**DATA:** ${data}

---

## Demanda

| Tipo de Modificação |   |
| :--- | :--- |
| BLOQUEIO | ${tipo === "BLOQUEIO" ? "☒" : "☐"} |
| RECLASSIFICAÇÃO | ${tipo === "RECLASSIFICACAO" ? "☒" : "☐"} |
| ADIÇÃO | ${tipo === "ADICAO" ? "☒" : "☐"} |
| CRIAÇÃO | ${tipo === "CRIACAO" ? "☒" : "☐"} |

**ELEMENTO-ITEM PROPOSTO:** ${elemento}  
**ELEMENTO-ITEM A SER SUBSTITUÍDO (CASO APLICÁVEL):** ${substituido}

---

## Contexto da demanda

${contexto}

---

## Solução encontrada

${solucao}

---

## Anexos

${pdfInput.files.length ? `📎 [Abrir PDF Anexo](anexos/ato-${n}.pdf)` : "Não houve"}
`;

  const token = prompt("Insira seu token GitHub (repo:write):");
  const basePath = `docs/atos/`;
  const path = `${basePath}ato-${n}.md`;

  // 1️⃣ Enviar o Markdown principal
  const payload = {
    message: `Criação automática do Ato ${n}`,
    content: btoa(markdown),
    branch: branch
  };

  const response = await fetch(`https://api.github.com/repos/${repo}/contents/${path}`, {
    method: "PUT",
    headers: {
      "Authorization": `token ${token}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify(payload)
  });

  // 2️⃣ Se houver PDF, enviar também
  if (pdfInput.files.length) {
    const file = pdfInput.files[0];
    const reader = new FileReader();
    reader.onload = async (event) => {
      const pdfBase64 = event.target.result.split(",")[1];
      const pdfPath = `docs/atos/anexos/ato-${n}.pdf`;
      const pdfPayload = {
        message: `Anexo PDF do Ato ${n}`,
        content: pdfBase64,
        branch: branch
      };

      await fetch(`https://api.github.com/repos/${repo}/contents/${pdfPath}`, {
        method: "PUT",
        headers: {
          "Authorization": `token ${token}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify(pdfPayload)
      });
    };
    reader.readAsDataURL(file);
  }

  if (response.ok) {
    alert("✅ Ato criado com sucesso! O site será atualizado automaticamente.");
  } else {
    const error = await response.json();
    alert("❌ Erro: " + (error.message || "verifique o token e permissões."));
  }
});
</script>
