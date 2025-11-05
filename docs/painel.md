# 🧾 Painel de Atualização – DCPPN

> Ferramenta interna para criação automática de **Atos DCPPN** diretamente no repositório GitHub.

---

<form id="novoAtoForm" style="display:flex;flex-direction:column;gap:10px;max-width:600px;">
  <label><b>Título do Ato</b></label>
  <input type="text" id="titulo" placeholder="Ex: Ato 144 – Modificação de Elemento-item" required>

  <label><b>Conteúdo (Markdown)</b></label>
  <textarea id="conteudo" rows="12" placeholder="Insira o conteúdo do ato em formato Markdown" required></textarea>

  <button type="submit" style="padding:10px;background-color:#1976D2;color:white;border:none;border-radius:5px;">Salvar Ato</button>
</form>

<script>
const repo = "pedrokenji0/dcppn"; // ajuste se o nome for outro
const branch = "main";            // ou "master" se seu repo usar esse nome

document.getElementById("novoAtoForm").addEventListener("submit", async (e) => {
  e.preventDefault();

  const titulo = document.getElementById("titulo").value.trim();
  const conteudo = document.getElementById("conteudo").value.trim();
  const filename = titulo.replace(/\s+/g, "-").toLowerCase() + ".md";
  const path = `docs/atos/${filename}`;

  const token = prompt("Insira seu token de acesso GitHub (repo:write):");

  const payload = {
    message: `Criação automática via painel: ${titulo}`,
    content: btoa(`# ${titulo}\n\n${conteudo}`),
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
    alert("✅ Ato criado com sucesso! O site será atualizado automaticamente em alguns minutos.");
  } else {
    const error = await response.json();
    alert("❌ Erro ao enviar: " + (error.message || "verifique o token e permissões."));
  }
});
</script>
