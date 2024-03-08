import tkinter as tk
from tkinter import filedialog
import os
import subprocess

def extract_and_save_chunks(file_path, output_folder, keyword, result_var):
    try:
        with open(file_path, 'r', encoding='latin-1') as file:
            content = file.read()

        chunk_size = 200
        output_index = 1
        combined_output = []

        for start_index in range(0, len(content), chunk_size):
            end_index = start_index + chunk_size
            chunk = content[start_index:end_index]

            # Verifica se a palavra-chave está presente no trecho
            if keyword.lower() in chunk.lower():
                output_file_path = os.path.join(output_folder, f"output_chunk_{output_index}.txt")
                output_index += 1

                with open(output_file_path, 'w', encoding='utf-8') as output_file:
                    output_file.write(chunk)

                combined_output.append(chunk)

        # Salva todos os trechos combinados em um único arquivo
        combined_file_path = os.path.join(output_folder, "output_combined.txt")
        with open(combined_file_path, 'w', encoding='utf-8') as combined_file:
            combined_file.write("\n".join(combined_output))

        # Abre o arquivo combinado usando o editor de texto padrão do sistema
        subprocess.run(['open', combined_file_path], check=True)  # Utiliza 'open' no MacOS, 'start' no Windows

        result_var.set(f"Trechos extraídos e arquivo combinado salvos na pasta: {output_folder}")
    except Exception as e:
        result_var.set(f"Erro ao processar o arquivo: {str(e)}")
        raise  # Adicionado para imprimir a traceback completa no console

def select_file():
    file_path = filedialog.askopenfilename(title="Selecionar Arquivo")
    entry_var.set(file_path)

def process_file():
    input_file = entry_var.get()
    if not input_file:
        result_var.set("Selecione um arquivo antes de processar.")
        return

    keyword = keyword_entry.get()

    output_folder = filedialog.askdirectory(title="Selecionar Pasta de Saída")
    if not output_folder:
        result_var.set("Operação cancelada.")
        return

    try:
        extract_and_save_chunks(input_file, output_folder, keyword, result_var)
    except Exception as e:
        print(f"Erro durante o processamento: {str(e)}")

# Interface gráfica
root = tk.Tk()
root.title("Extrair Trechos em Torno de Palavra-chave")

entry_var = tk.StringVar()
result_var = tk.StringVar()

tk.Label(root, text="Selecione o arquivo:").pack(pady=10)
tk.Entry(root, textvariable=entry_var, state="readonly", width=40).pack(pady=5)
tk.Button(root, text="Selecionar Arquivo", command=select_file).pack(pady=5)

tk.Label(root, text="Palavra-chave para extrair trechos (case-insensitive):").pack(pady=5)
keyword_entry = tk.Entry(root, width=40)
keyword_entry.pack(pady=5)

process_button = tk.Button(root, text="Enter", command=process_file)
process_button.pack(pady=10)

tk.Label(root, textvariable=result_var, fg="green").pack()

root.mainloop()
