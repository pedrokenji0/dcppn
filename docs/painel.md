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

  <label><b>Anexos</b></label>
  <input type="text" id="anexos" placeholder="Ex: Não houve / PDF anexo">

  <button type="submit" style="padding:10px;background-color:#1976D2;color:white;border:none;border-radius:5px;">Gerar e Salvar Ato</button>
</form>

<script>
const repo = "pedrokenji0/dcppn";
const branch = "main";

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
  const anexos = document.getElementById("anexos").value.trim();

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

${anexos}
`;

  const filename = `ato-${n}.md`;
  const path = `docs/atos/${filename}`;
  const token = prompt("Insira seu token GitHub (repo:write):");

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

  if (response.ok) {
    alert("✅ Ato criado com sucesso! O site será atualizado automaticamente.");
  } else {
    const error = await response.json();
    alert("❌ Erro: " + (error.message || "verifique o token e permissões."));
  }
});
</script>

