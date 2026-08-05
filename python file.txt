import tkinter as tk
from tkinter import ttk
import random

# -------- Algorithms -------- #

def first_fit(blocks, processes):
    alloc = [-1] * len(processes)
    for i in range(len(processes)):
        for j in range(len(blocks)):
            if blocks[j] >= processes[i]:
                alloc[i] = j
                blocks[j] -= processes[i]
                break
    return alloc

def best_fit(blocks, processes):
    alloc = [-1] * len(processes)
    for i in range(len(processes)):
        best = -1
        for j in range(len(blocks)):
            if blocks[j] >= processes[i]:
                if best == -1 or blocks[j] < blocks[best]:
                    best = j
        if best != -1:
            alloc[i] = best
            blocks[best] -= processes[i]
    return alloc

def worst_fit(blocks, processes):
    alloc = [-1] * len(processes)
    for i in range(len(processes)):
        worst = -1
        for j in range(len(blocks)):
            if blocks[j] >= processes[i]:
                if worst == -1 or blocks[j] > blocks[worst]:
                    worst = j
        if worst != -1:
            alloc[i] = worst
            blocks[worst] -= processes[i]
    return alloc

# -------- Utility -------- #

def calculate_waste(blocks, alloc, processes):
    temp = blocks.copy()

    for i in range(len(processes)):
        if alloc[i] != -1:
            temp[alloc[i]] -= processes[i]

    waste = 0
    for i in range(len(blocks)):
        if temp[i] != blocks[i]:
            waste += temp[i]

    return waste

def fill_table(table, processes, alloc):
    for row in table.get_children():
        table.delete(row)

    for i in range(len(processes)):
        block = f"B{alloc[i]+1}" if alloc[i] != -1 else "Not Allocated"
        table.insert("", "end", values=(f"P{i+1}", processes[i], block))

def generate():
    try:
        nb = int(block_entry.get())
        np = int(process_entry.get())
    except:
        return None, None

    blocks = [random.randint(100, 400) for _ in range(nb)]
    processes = [random.randint(50, 250) for _ in range(np)]

    blocks_label.config(text=f"Memory Blocks: {blocks}")
    process_label.config(text=f"Processes: {processes}")

    return blocks, processes

# -------- UI Logic -------- #

def hide_all():
    tables_frame.pack_forget()
    single_frame.pack_forget()
    result_label.config(text="")
    best_label.config(text="")

def show_single(title, alloc, processes):
    hide_all()
    single_title.config(text=title)
    single_frame.pack(pady=15)
    fill_table(single_table, processes, alloc)

def run_first():
    b, p = generate()
    if b:
        show_single("First Fit", first_fit(b.copy(), p), p)

def run_best():
    b, p = generate()
    if b:
        show_single("Best Fit", best_fit(b.copy(), p), p)

def run_worst():
    b, p = generate()
    if b:
        show_single("Worst Fit", worst_fit(b.copy(), p), p)

def run_all():
    blocks, processes = generate()
    if not blocks:
        return

    hide_all()
    tables_frame.pack(pady=15)

    ff = first_fit(blocks.copy(), processes)
    bf = best_fit(blocks.copy(), processes)
    wf = worst_fit(blocks.copy(), processes)

    fill_table(table_ff, processes, ff)
    fill_table(table_bf, processes, bf)
    fill_table(table_wf, processes, wf)

    ff_w = calculate_waste(blocks, ff, processes)
    bf_w = calculate_waste(blocks, bf, processes)
    wf_w = calculate_waste(blocks, wf, processes)

    best_algo = "All Equal" if ff_w == bf_w == wf_w else min(
        {"First Fit": ff_w, "Best Fit": bf_w, "Worst Fit": wf_w},
        key=lambda x: {"First Fit": ff_w, "Best Fit": bf_w, "Worst Fit": wf_w}[x]
    )

    result_label.config(
        text=f"Memory Wastage\nFirst Fit = {ff_w} | Best Fit = {bf_w} | Worst Fit = {wf_w}",
        fg="#333"
    )

    best_label.config(
        text=f"Most Efficient: {best_algo}",
        fg="#2E7D32"
    )

# -------- GUI -------- #

root = tk.Tk()
root.title("Memory Allocation Simulator")
root.geometry("900x600")
root.configure(bg="#F5F3FF")   # soft background

# STYLE (SOFT HEADERS)
style = ttk.Style()

style.configure("Treeview.Heading",
                font=("Segoe UI", 10, "bold"),
                background="#E8E6FF",
                foreground="#2C2C2C")

style.map("Treeview.Heading",
          background=[("active", "#DDDBFF")])

# Main frame
main_frame = tk.Frame(root, bg="#F5F3FF")
main_frame.pack(expand=True)

tk.Label(
    main_frame,
    text="Memory Allocation Simulator",
    font=("Segoe UI", 28, "bold"),
    bg="#F5F3FF",
    fg="#3A3A3A"
).pack(pady=20)

# Inputs
input_frame = tk.Frame(main_frame, bg="#F5F3FF")
input_frame.pack()

tk.Label(input_frame, text="Blocks:", bg="#F5F3FF").grid(row=0, column=0, padx=10)
block_entry = tk.Entry(input_frame, width=10)
block_entry.grid(row=0, column=1)

tk.Label(input_frame, text="Processes:", bg="#F5F3FF").grid(row=0, column=2, padx=10)
process_entry = tk.Entry(input_frame, width=10)
process_entry.grid(row=0, column=3)

# Buttons (SOFT LIKE COMPARE ALL)
btn_frame = tk.Frame(main_frame, bg="#F5F3FF")
btn_frame.pack(pady=10)

tk.Button(btn_frame, text="First Fit", command=run_first,
          width=12, bg="#E8E6FF", bd=0).grid(row=0, column=0, padx=5)

tk.Button(btn_frame, text="Best Fit", command=run_best,
          width=12, bg="#E8E6FF", bd=0).grid(row=0, column=1, padx=5)

tk.Button(btn_frame, text="Worst Fit", command=run_worst,
          width=12, bg="#E8E6FF", bd=0).grid(row=0, column=2, padx=5)

tk.Button(btn_frame, text="Compare All", command=run_all,
          width=14, bg="#EDE7FF", bd=0).grid(row=0, column=3, padx=5)

# Tables
tables_frame = tk.Frame(main_frame, bg="#F5F3FF")

def create_table(title):
    frame = tk.Frame(tables_frame, bg="#F5F3FF")
    frame.pack(side="left", padx=10)

    tk.Label(
        frame,
        text=title,
        font=("Segoe UI", 13, "bold"),
        bg="#EAF7F0",
        fg="#2C2C2C",
        padx=10,
        pady=5,
        relief="solid",
        borderwidth=1
    ).pack(pady=5, fill="x")

    table = ttk.Treeview(frame, columns=("P", "Size", "Block"),
                         show="headings", height=8)

    table.heading("P", text="PROCESSES")
    table.heading("Size", text="SIZES")
    table.heading("Block", text="BLOCK ALLOCATED")

    table.column("P", width=100, anchor="center")
    table.column("Size", width=100, anchor="center")
    table.column("Block", width=140, anchor="center")

    table.pack()
    return table

table_ff = create_table("First Fit")
table_bf = create_table("Best Fit")
table_wf = create_table("Worst Fit")

# Single table
single_frame = tk.Frame(main_frame, bg="#F5F3FF")

single_title = tk.Label(
    single_frame,
    font=("Segoe UI", 14, "bold"),
    bg="#EAF7F0",
    fg="#2C2C2C",
    padx=10,
    pady=5,
    relief="solid",
    borderwidth=1
)
single_title.pack(pady=5, fill="x")

single_table = ttk.Treeview(single_frame,
                           columns=("P", "Size", "Block"),
                           show="headings", height=8)

single_table.heading("P", text="PROCESSES")
single_table.heading("Size", text="SIZES")
single_table.heading("Block", text="BLOCK ALLOCATED")

single_table.column("P", width=120, anchor="center")
single_table.column("Size", width=120, anchor="center")
single_table.column("Block", width=160, anchor="center")

single_table.pack()

# Info labels
blocks_label = tk.Label(main_frame, text="", bg="#F5F3FF")
blocks_label.pack()

process_label = tk.Label(main_frame, text="", bg="#F5F3FF")
process_label.pack()

result_label = tk.Label(main_frame, text="", bg="#F5F3FF",
                        font=("Segoe UI", 10))
result_label.pack(pady=5)

best_label = tk.Label(main_frame, text="", bg="#F5F3FF",
                      font=("Segoe UI", 11, "bold"))
best_label.pack()

root.mainloop()