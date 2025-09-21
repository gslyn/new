import tkinter as tk
from tkinter import messagebox
import json
import os

# ---------- Classe Gestor de Áreas ----------
class GestorAreas:
    def __init__(self, arquivo="areas.json"):
        self.arquivo = arquivo
        self.areas = []
        self.carregar_areas()

    def carregar_areas(self):
        if os.path.exists(self.arquivo):
            try:
                with open(self.arquivo, "r", encoding="utf-8") as f:
                    data = json.load(f)
            except Exception:
                self.areas = []
                return
        else:
            self.areas = []
            return

        if not isinstance(data, list):
            self.areas = []
            return

        migrated = False
        for item in data:
            if not isinstance(item, dict):
                continue
            for key in ("nome","localizacao","preco","eletricidade","agua","despejo_cinzas","despejo_negras","visitado"):
                if key not in item:
                    if key == "preco":
                        item[key] = 0.0
                    elif key in ("eletricidade","agua","despejo_cinzas","despejo_negras","visitado"):
                        item[key] = False
                    else:
                        item[key] = ""
                    migrated = True
        self.areas = data
        if migrated:
            self.salvar_areas()

    def salvar_areas(self):
        with open(self.arquivo, "w", encoding="utf-8") as f:
            json.dump(self.areas, f, indent=4, ensure_ascii=False)

    def adicionar_area(self, nome, localizacao, preco, eletricidade, agua, cinzas, negras):
        # Evitar duplicados
        for area in self.areas:
            if area["nome"].lower() == nome.lower() and area["localizacao"].lower() == localizacao.lower():
                return False  # já existe

        try:
            preco = float(str(preco).replace(",", "."))
        except Exception:
            preco = 0.0
        nova_area = {
            "nome": nome,
            "localizacao": localizacao,
            "preco": preco,
            "eletricidade": bool(eletricidade),
            "agua": bool(agua),
            "despejo_cinzas": bool(cinzas),
            "despejo_negras": bool(negras),
            "visitado": False
        }
        self.areas.append(nova_area)
        self.salvar_areas()
        return True

    def remover_area(self, indice):
        if 0 <= indice < len(self.areas):
            self.areas.pop(indice)
            self.salvar_areas()
            return True
        return False

    def alternar_visitado(self, indice):
        if 0 <= indice < len(self.areas):
            self.areas[indice]["visitado"] = not self.areas[indice]["visitado"]
            self.salvar_areas()
            return True
        return False


# ---------- Classe da Interface ----------
class App:
    def __init__(self, root):
        self.gestor = GestorAreas()

        # --- Cores personalizadas ---
        cor_fundo = "#e6f2ff"      # azul clarinho
        cor_botao = "#4da6ff"      # azul médio
        cor_botao_texto = "white"
        cor_lista_fundo = "white"
        cor_lista_texto = "black"

        root.title("Aplicação para Autocaravanistas")
        root.geometry("720x600")
        root.configure(bg=cor_fundo)
        root.resizable(False, False)

        # --- Formulário ---
        frame_form = tk.Frame(root, bg=cor_fundo)
        frame_form.pack(pady=10)

        tk.Label(frame_form, text="Nome:", bg=cor_fundo).grid(row=0, column=0, sticky="w")
        self.entry_nome = tk.Entry(frame_form, width=30)
        self.entry_nome.grid(row=0, column=1, padx=5)

        tk.Label(frame_form, text="Localização:", bg=cor_fundo).grid(row=1, column=0, sticky="w")
        self.entry_local = tk.Entry(frame_form, width=30)
        self.entry_local.grid(row=1, column=1, padx=5)

        tk.Label(frame_form, text="Preço (€):", bg=cor_fundo).grid(row=2, column=0, sticky="w")
        self.entry_preco = tk.Entry(frame_form, width=12)
        self.entry_preco.grid(row=2, column=1, sticky="w", padx=5)

        # Serviços disponíveis
        self.var_eletricidade = tk.BooleanVar()
        tk.Checkbutton(frame_form, text="Eletricidade ⚡", variable=self.var_eletricidade, bg=cor_fundo).grid(row=3, column=0, sticky="w")

        self.var_agua = tk.BooleanVar()
        tk.Checkbutton(frame_form, text="Água 💧", variable=self.var_agua, bg=cor_fundo).grid(row=3, column=1, sticky="w")

        self.var_cinzas = tk.BooleanVar()
        tk.Checkbutton(frame_form, text="Despejo Águas Cinzas ♻", variable=self.var_cinzas, bg=cor_fundo).grid(row=4, column=0, sticky="w")

        self.var_negras = tk.BooleanVar()
        tk.Checkbutton(frame_form, text="Despejo Águas Negras 🚽", variable=self.var_negras, bg=cor_fundo).grid(row=4, column=1, sticky="w")

        tk.Button(frame_form, text="Adicionar Área", command=self.adicionar,
                  bg=cor_botao, fg=cor_botao_texto).grid(row=5, columnspan=2, pady=10)

        # --- Lista ---
        self.lista = tk.Listbox(root, width=110, height=15, font=("Arial", 10),
                                bg=cor_lista_fundo, fg=cor_lista_texto)
        self.lista.pack(pady=10)

        # --- Botões de ação ---
        frame_botoes = tk.Frame(root, bg=cor_fundo)
        frame_botoes.pack(pady=5)

        tk.Button(frame_botoes, text="Remover Selecionada", command=self.remover,
                  bg=cor_botao, fg=cor_botao_texto).grid(row=0, column=0, padx=10)
        tk.Button(frame_botoes, text="Alternar Visitado", command=self.alternar_visitado,
                  bg=cor_botao, fg=cor_botao_texto).grid(row=0, column=1, padx=10)

        # --- Rodapé com contagem ---
        self.label_status = tk.Label(root, text="", bg=cor_fundo, font=("Arial", 10, "bold"))
        self.label_status.pack(pady=5)

        # Carregar lista inicial
        self.atualizar_lista()

    def atualizar_lista(self):
        self.lista.delete(0, tk.END)
        # Ordenar por nome
        for a in sorted(self.gestor.areas, key=lambda x: x["nome"].lower()):
            status = "✔ Visitado" if a.get("visitado", False) else "✗ Não visitado"
            eletricidade = "⚡" if a.get("eletricidade", False) else ""
            agua = "💧" if a.get("agua", False) else ""
            cinzas = "♻" if a.get("despejo_cinzas", False) else ""
            negras = "🚽" if a.get("despejo_negras", False) else ""
            preco = a.get("preco", 0.0)
            preco_str = f"{float(preco):.2f}" if isinstance(preco, (int,float)) else "0.00"
            self.lista.insert(
                tk.END,
                f"{a.get('nome','Área sem nome')} - {a.get('localizacao','')} | {preco_str}€/noite {eletricidade}{agua}{cinzas}{negras} | {status}"
            )

        # Atualizar rodapé
        total = len(self.gestor.areas)
        visitadas = sum(1 for a in self.gestor.areas if a.get("visitado", False))
        self.label_status.config(text=f"Áreas registadas: {total} | Visitadas: {visitadas}")

    def adicionar(self):
        nome = self.entry_nome.get().strip()
        local = self.entry_local.get().strip()
        preco = self.entry_preco.get().strip()

        if not nome or not local or not preco:
            messagebox.showwarning("Aviso", "Preencha todos os campos obrigatórios.")
            return

        try:
            preco_val = float(preco.replace(",", "."))
        except ValueError:
            messagebox.showwarning("Aviso", "O preço deve ser um número.")
            return

        sucesso = self.gestor.adicionar_area(
            nome,
            local,
            preco_val,
            self.var_eletricidade.get(),
            self.var_agua.get(),
            self.var_cinzas.get(),
            self.var_negras.get()
        )

        if not sucesso:
            messagebox.showwarning("Aviso", "Já existe uma área com esse nome e localização.")
            return

        messagebox.showinfo("Sucesso", "Área adicionada com sucesso!")

        # Reset campos
        self.entry_nome.delete(0, tk.END)
        self.entry_local.delete(0, tk.END)
        self.entry_preco.delete(0, tk.END)
        self.var_eletricidade.set(False)
        self.var_agua.set(False)
        self.var_cinzas.set(False)
        self.var_negras.set(False)

        self.atualizar_lista()

    def remover(self):
        selecionado = self.lista.curselection()
        if selecionado:
            if self.gestor.remover_area(selecionado[0]):
                messagebox.showinfo("Sucesso", "Área removida com sucesso.")
                self.atualizar_lista()
        else:
            messagebox.showwarning("Aviso", "Selecione uma área para remover.")

    def alternar_visitado(self):
        selecionado = self.lista.curselection()
        if selecionado:
            if self.gestor.alternar_visitado(selecionado[0]):
                self.atualizar_lista()
        else:
            messagebox.showwarning("Aviso", "Selecione uma área para alternar o estado.")


# ---------- Execução ----------
if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()
