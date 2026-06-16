import tkinter as tk
from tkinter import ttk, messagebox
import math
import matplotlib
matplotlib.use('TkAgg')
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import numpy as np

# FUNGSI KEANGGOTAAN

def trimf(x, a, b, c):
    if x <= a or x >= c:
        return 0.0
    if x <= b:
        return (x - a) / (b - a)
    return (c - x) / (c - b)

def trapmf_down(x, a, b):
    if x <= a:
        return 1.0
    if x >= b:
        return 0.0
    return (b - x) / (b - a)

def trapmf_up(x, a, b):
    if x <= a:
        return 0.0
    if x >= b:
        return 1.0
    return (x - a) / (b - a)

# FUZZIFIKASI

def fuzzify_umur(u):
    return {
        'muda':       trapmf_down(u, 20, 40),
        'paruh_baya': trimf(u, 30, 47, 62),
        'tua':        trapmf_up(u, 50, 70),
    }

def fuzzify_bmi(b):
    return {
        'kurus':  trapmf_down(b, 10, 20),
        'normal': trimf(b, 18, 24, 30),
        'gemuk':  trapmf_up(b, 27, 40),
    }

def fuzzify_gula(g):
    return {
        'normal':       trapmf_down(g, 70, 105),
        'pra_diabetes': trimf(g, 90, 118, 145),
        'tinggi':       trapmf_up(g, 126, 200),
    }

def fuzzify_aktivitas(a):
    return {
        'rendah': trapmf_down(a, 0, 4),
        'sedang': trimf(a, 2, 5, 8),
        'tinggi': trapmf_up(a, 6, 10),
    }


# FUNGSI INVERS OUTPUT (Tsukamoto)


def inverse_output(alpha, himpunan):
    if alpha <= 0:
        return None
    if himpunan == 'rendah':
        return 40 - alpha * 40
    elif himpunan == 'sedang':
        return 30 + alpha * 40
    elif himpunan == 'tinggi':
        return 60 + alpha * 40
    return None


# RULE BASE

RULES = [
    ('muda',       'kurus',  'normal',       'tinggi', 'rendah'),
    ('muda',       'normal', 'normal',       'sedang', 'rendah'),
    ('muda',       'gemuk',  'pra_diabetes', 'rendah', 'sedang'),
    ('paruh_baya', 'normal', 'normal',       'tinggi', 'rendah'),
    ('paruh_baya', 'gemuk',  'pra_diabetes', 'sedang', 'sedang'),
    ('paruh_baya', 'gemuk',  'tinggi',       'rendah', 'tinggi'),
    ('tua',        'normal', 'pra_diabetes', 'sedang', 'sedang'),
    ('tua',        'gemuk',  'tinggi',       'rendah', 'tinggi'),
    ('tua',        'kurus',  'normal',       'rendah', 'sedang'),
    ('muda',       'gemuk',  'tinggi',       'rendah', 'sedang'),
    ('paruh_baya', 'normal', 'pra_diabetes', 'rendah', 'sedang'),
    ('tua',        'normal', 'normal',       'tinggi', 'rendah'),
    ('paruh_baya', 'kurus',  'normal',       'tinggi', 'rendah'),
]


# INFERENSI TSUKAMOTO

def tsukamoto(umur, bmi, gula, aktivitas):
    mu_u = fuzzify_umur(umur)
    mu_b = fuzzify_bmi(bmi)
    mu_g = fuzzify_gula(gula)
    mu_a = fuzzify_aktivitas(aktivitas)

    detail_rules = []
    total_az = 0.0
    total_a  = 0.0

    for i, (r_u, r_b, r_g, r_a, r_out) in enumerate(RULES):
        alpha = min(
            mu_u.get(r_u, 0),
            mu_b.get(r_b, 0),
            mu_g.get(r_g, 0),
            mu_a.get(r_a, 0),
        )
        z = inverse_output(alpha, r_out)

        rule_text = (f"IF Umur={r_u.replace('_',' ')} AND BMI={r_b} "
                     f"AND Gula={r_g.replace('_',' ')} AND Aktivitas={r_a} THEN {r_out}")

        detail_rules.append({
            'no':    i + 1,
            'rule':  rule_text,
            'alpha': alpha,
            'z':     z,
            'aktif': alpha > 0,
        })

        if z is not None and alpha > 0:
            total_az += alpha * z
            total_a  += alpha

    if total_a == 0:
        return 0, mu_u, mu_b, mu_g, mu_a, detail_rules

    z_crisp = total_az / total_a
    return z_crisp, mu_u, mu_b, mu_g, mu_a, detail_rules


def interpret(z):
    if z < 35:
        return "RENDAH", "#27ae60", "✓ Risiko diabetes rendah. Pertahankan gaya hidup sehat!"
    elif z < 65:
        return "SEDANG", "#f39c12", "⚠ Risiko diabetes sedang. Perlu pemantauan rutin."
    else:
        return "TINGGI", "#e74c3c", "✗ Risiko diabetes tinggi! Segera konsultasi dokter."


# APLIKASI GUI

class FuzzyDiabetesApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistem Penentuan Risiko Diabetes — Fuzzy Tsukamoto")
        self.geometry("1100x780")
        self.configure(bg="#0f172a")
        self.resizable(True, True)

        self._build_header()
        self._build_main()
        self._build_footer()

    def _build_header(self):
        hdr = tk.Frame(self, bg="#1e293b", pady=12)
        hdr.pack(fill='x')

        tk.Label(hdr, text="🩺  SISTEM PENENTUAN RISIKO DIABETES",
                 font=("Segoe UI", 18, "bold"),
                 fg="#38bdf8", bg="#1e293b").pack()

        tk.Label(hdr, text="Metode Inferensi Fuzzy Tsukamoto  |  Kelompok 7",
                 font=("Segoe UI", 10),
                 fg="#94a3b8", bg="#1e293b").pack()

    def _build_main(self):
        body = tk.Frame(self, bg="#0f172a")
        body.pack(fill='both', expand=True, padx=20, pady=10)

        left = tk.Frame(body, bg="#1e293b", bd=0, relief='flat')
        left.pack(side='left', fill='y', padx=(0, 10))
        self._build_inputs(left)

        right = tk.Frame(body, bg="#0f172a")
        right.pack(side='left', fill='both', expand=True)
        self._build_output(right)

    def _build_inputs(self, parent):
        tk.Label(parent, text="DATA PASIEN",
                 font=("Segoe UI", 11, "bold"),
                 fg="#38bdf8", bg="#1e293b").pack(pady=(16, 4))

        tk.Frame(parent, bg="#334155", height=1, width=260).pack(pady=4)

        self.vars = {}

        fields = [
            ("Umur (tahun)",          "umur",      15, 100, 45,  "tahun"),
            ("BMI (kg/m²)",           "bmi",       10,  50, 27,  "kg/m²"),
            ("Gula Darah (mg/dL)",    "gula",      70, 300, 130, "mg/dL"),
            ("Aktivitas Fisik (0–10)","aktivitas",  0,  10,  3,  "/10"),
        ]

        for label, key, mn, mx, default, unit in fields:
            self._input_row(parent, label, key, mn, mx, default, unit)

        tk.Frame(parent, bg="#334155", height=1, width=260).pack(pady=10)

        tk.Button(parent, text="⚡  HITUNG RISIKO",
                  font=("Segoe UI", 12, "bold"),
                  bg="#38bdf8", fg="#0f172a",
                  activebackground="#0ea5e9",
                  relief='flat', cursor='hand2',
                  padx=20, pady=10,
                  command=self._hitung).pack(pady=6, padx=20, fill='x')

        tk.Button(parent, text="↺  Reset",
                  font=("Segoe UI", 9),
                  bg="#334155", fg="#94a3b8",
                  activebackground="#475569",
                  relief='flat', cursor='hand2',
                  padx=10, pady=6,
                  command=self._reset).pack(pady=2, padx=20, fill='x')

    def _input_row(self, parent, label, key, mn, mx, default, unit):
        frame = tk.Frame(parent, bg="#1e293b", padx=16, pady=6)
        frame.pack(fill='x')

        tk.Label(frame, text=label, font=("Segoe UI", 9),
                 fg="#94a3b8", bg="#1e293b", anchor='w').pack(fill='x')

        row = tk.Frame(frame, bg="#1e293b")
        row.pack(fill='x')

        var = tk.DoubleVar(value=default)
        self.vars[key] = var

        res = 0.1 if key == 'bmi' else 1
        scale = tk.Scale(row, from_=mn, to=mx,
                         orient='horizontal', variable=var,
                         resolution=res,
                         bg="#1e293b", fg="#e2e8f0",
                         troughcolor="#334155",
                         activebackground="#38bdf8",
                         highlightthickness=0,
                         showvalue=False, length=160,
                         command=lambda v, k=key: self._update_entry(k))
        scale.pack(side='left')

        entry = tk.Entry(row, width=6, font=("Segoe UI", 10, "bold"),
                         bg="#0f172a", fg="#38bdf8",
                         insertbackground="#38bdf8",
                         relief='flat', justify='center')
        entry.insert(0, str(default))
        entry.pack(side='left', padx=6)
        entry.bind("<Return>",   lambda e, k=key: self._entry_to_scale(k))
        entry.bind("<FocusOut>", lambda e, k=key: self._entry_to_scale(k))

        tk.Label(row, text=unit, font=("Segoe UI", 8),
                 fg="#64748b", bg="#1e293b").pack(side='left')

        setattr(self, f"entry_{key}", entry)
        setattr(self, f"scale_{key}", scale)

    def _update_entry(self, key):
        entry = getattr(self, f"entry_{key}")
        entry.delete(0, 'end')
        val = self.vars[key].get()
        entry.insert(0, f"{val:.1f}" if key == 'bmi' else str(int(val)))

    def _entry_to_scale(self, key):
        entry = getattr(self, f"entry_{key}")
        try:
            self.vars[key].set(float(entry.get()))
        except:
            pass

    def _build_output(self, parent):
        self.result_frame = tk.Frame(parent, bg="#1e293b", pady=16)
        self.result_frame.pack(fill='x', pady=(0, 10))

        self.lbl_status = tk.Label(self.result_frame, text="—",
                                   font=("Segoe UI", 36, "bold"),
                                   fg="#334155", bg="#1e293b")
        self.lbl_status.pack()

        self.lbl_skor = tk.Label(self.result_frame,
                                 text="Masukkan data pasien lalu klik Hitung Risiko",
                                 font=("Segoe UI", 11), fg="#64748b", bg="#1e293b")
        self.lbl_skor.pack()

        self.lbl_info = tk.Label(self.result_frame, text="",
                                 font=("Segoe UI", 10), fg="#94a3b8", bg="#1e293b")
        self.lbl_info.pack(pady=4)

        nb = ttk.Notebook(parent)
        nb.pack(fill='both', expand=True)

        style = ttk.Style()
        style.theme_use('clam')
        style.configure("TNotebook", background="#0f172a", borderwidth=0)
        style.configure("TNotebook.Tab", background="#1e293b", foreground="#94a3b8",
                        font=("Segoe UI", 9, "bold"), padding=(12, 6))
        style.map("TNotebook.Tab",
                  background=[("selected", "#334155")],
                  foreground=[("selected", "#38bdf8")])

        self.tab_grafik = tk.Frame(nb, bg="#0f172a")
        nb.add(self.tab_grafik, text="  📊 Grafik Fungsi Keanggotaan  ")

        self.tab_rules = tk.Frame(nb, bg="#0f172a")
        nb.add(self.tab_rules, text="  📋 Detail Inferensi Tsukamoto  ")

        self._build_grafik_placeholder()
        self._build_rules_table()

    def _build_grafik_placeholder(self):
        tk.Label(self.tab_grafik,
                 text="Klik 'Hitung Risiko' untuk melihat grafik",
                 font=("Segoe UI", 11), fg="#475569", bg="#0f172a").pack(expand=True)

    def _build_rules_table(self):
        frame = tk.Frame(self.tab_rules, bg="#0f172a")
        frame.pack(fill='both', expand=True, padx=8, pady=8)

        cols = ("No", "Rule", "α (Alpha)", "z (Crisp)", "Status")
        self.tree = ttk.Treeview(frame, columns=cols, show='headings', height=14)

        style = ttk.Style()
        style.configure("Treeview",
                        background="#1e293b", foreground="#e2e8f0",
                        fieldbackground="#1e293b", rowheight=26,
                        font=("Segoe UI", 9))
        style.configure("Treeview.Heading",
                        background="#334155", foreground="#38bdf8",
                        font=("Segoe UI", 9, "bold"))
        style.map("Treeview", background=[("selected", "#334155")])

        self.tree.heading("No",        text="No")
        self.tree.heading("Rule",      text="Fuzzy Rule")
        self.tree.heading("α (Alpha)", text="α (Alpha)")
        self.tree.heading("z (Crisp)", text="z Output")
        self.tree.heading("Status",    text="Status")

        self.tree.column("No",        width=35,  anchor='center')
        self.tree.column("Rule",      width=430, anchor='w')
        self.tree.column("α (Alpha)", width=80,  anchor='center')
        self.tree.column("z (Crisp)", width=80,  anchor='center')
        self.tree.column("Status",    width=70,  anchor='center')

        sb = ttk.Scrollbar(frame, orient='vertical', command=self.tree.yview)
        self.tree.configure(yscrollcommand=sb.set)
        self.tree.pack(side='left', fill='both', expand=True)
        sb.pack(side='right', fill='y')

        self.lbl_defuzz = tk.Label(self.tab_rules, text="",
                                   font=("Segoe UI", 10, "bold"),
                                   fg="#38bdf8", bg="#0f172a")
        self.lbl_defuzz.pack(pady=6)

    def _hitung(self):
        try:
            umur = float(self.vars['umur'].get())
            bmi  = float(self.vars['bmi'].get())
            gula = float(self.vars['gula'].get())
            aktv = float(self.vars['aktivitas'].get())
        except:
            messagebox.showerror("Error", "Input tidak valid!")
            return

        z, mu_u, mu_b, mu_g, mu_a, detail = tsukamoto(umur, bmi, gula, aktv)
        label, color, info = interpret(z)

        self.lbl_status.config(text=f"RISIKO  {label}", fg=color)
        self.lbl_skor.config(text=f"Skor Defuzzifikasi: {z:.2f} / 100", fg="#e2e8f0")
        self.lbl_info.config(text=info, fg=color)

        for item in self.tree.get_children():
            self.tree.delete(item)

        aktif_count = 0
        total_az, total_a = 0.0, 0.0

        for d in detail:
            tag   = 'aktif' if d['aktif'] else 'nonaktif'
            z_str = f"{d['z']:.4f}" if d['z'] is not None else "—"
            st    = "✓ Aktif" if d['aktif'] else "✗"
            self.tree.insert('', 'end',
                             values=(d['no'], d['rule'],
                                     f"{d['alpha']:.4f}", z_str, st),
                             tags=(tag,))
            if d['aktif']:
                aktif_count += 1
                total_az += d['alpha'] * d['z']
                total_a  += d['alpha']

        self.tree.tag_configure('aktif',    background='#1a3a2a', foreground='#4ade80')
        self.tree.tag_configure('nonaktif', background='#1e293b', foreground='#64748b')

        self.lbl_defuzz.config(
            text=(f"Defuzzifikasi → Σ(α×z) / Σα  =  "
                  f"{total_az:.4f} / {total_a:.4f}  =  {z:.4f}   "
                  f"[{aktif_count} rule aktif dari {len(detail)}]")
        )

        self._update_grafik(umur, bmi, gula, aktv, mu_u, mu_b, mu_g, mu_a, z)

    def _update_grafik(self, umur, bmi, gula, aktv, mu_u, mu_b, mu_g, mu_a, z_result):
        for w in self.tab_grafik.winfo_children():
            w.destroy()

        fig, axes = plt.subplots(1, 5, figsize=(13, 2.8))
        fig.patch.set_facecolor('#0f172a')
        plt.subplots_adjust(wspace=0.45, left=0.04, right=0.98, top=0.82, bottom=0.22)

        BLUE   = '#38bdf8'
        GREEN  = '#4ade80'
        ORANGE = '#fb923c'
        RED    = '#f87171'
        GRID   = '#1e3a5f'
        AX_BG  = '#111827'

        def style_ax(ax, title):
            ax.set_facecolor(AX_BG)
            ax.set_title(title, color='#94a3b8', fontsize=8, pad=5)
            ax.tick_params(colors='#475569', labelsize=7)
            for spine in ax.spines.values():
                spine.set_edgecolor('#1e3a5f')
            ax.grid(True, color=GRID, linewidth=0.4, linestyle='--', alpha=0.6)

        # Umur
        ax = axes[0]
        x = np.linspace(0, 100, 400)
        ax.plot(x, [trapmf_down(v, 20, 40) for v in x], color=BLUE,   lw=1.5, label='Muda')
        ax.plot(x, [trimf(v, 30, 47, 62)   for v in x], color=GREEN,  lw=1.5, label='Paruh Baya')
        ax.plot(x, [trapmf_up(v, 50, 70)   for v in x], color=ORANGE, lw=1.5, label='Tua')
        ax.axvline(umur, color='white', lw=1, linestyle='--', alpha=0.7)
        ax.scatter([umur]*3, [mu_u['muda'], mu_u['paruh_baya'], mu_u['tua']],
                   color='white', zorder=5, s=18)
        style_ax(ax, f"Umur  [{umur:.0f} thn]")
        ax.legend(fontsize=6, labelcolor='#94a3b8', facecolor='#1e293b',
                  edgecolor='none', loc='upper right')

        # BMI
        ax = axes[1]
        x = np.linspace(10, 50, 400)
        ax.plot(x, [trapmf_down(v, 10, 20) for v in x], color=BLUE,   lw=1.5, label='Kurus')
        ax.plot(x, [trimf(v, 18, 24, 30)   for v in x], color=GREEN,  lw=1.5, label='Normal')
        ax.plot(x, [trapmf_up(v, 27, 40)   for v in x], color=ORANGE, lw=1.5, label='Gemuk')
        ax.axvline(bmi, color='white', lw=1, linestyle='--', alpha=0.7)
        ax.scatter([bmi]*3, [mu_b['kurus'], mu_b['normal'], mu_b['gemuk']],
                   color='white', zorder=5, s=18)
        style_ax(ax, f"BMI  [{bmi:.1f}]")
        ax.legend(fontsize=6, labelcolor='#94a3b8', facecolor='#1e293b',
                  edgecolor='none', loc='upper right')

        # Gula Darah
        ax = axes[2]
        x = np.linspace(70, 300, 400)
        ax.plot(x, [trapmf_down(v, 70, 105)  for v in x], color=BLUE,  lw=1.5, label='Normal')
        ax.plot(x, [trimf(v, 90, 118, 145)   for v in x], color=GREEN, lw=1.5, label='Pra-DM')
        ax.plot(x, [trapmf_up(v, 126, 200)   for v in x], color=RED,   lw=1.5, label='Tinggi')
        ax.axvline(gula, color='white', lw=1, linestyle='--', alpha=0.7)
        ax.scatter([gula]*3, [mu_g['normal'], mu_g['pra_diabetes'], mu_g['tinggi']],
                   color='white', zorder=5, s=18)
        style_ax(ax, f"Gula Darah  [{gula:.0f} mg/dL]")
        ax.legend(fontsize=6, labelcolor='#94a3b8', facecolor='#1e293b',
                  edgecolor='none', loc='upper right')

        # Aktivitas
        ax = axes[3]
        x = np.linspace(0, 10, 400)
        ax.plot(x, [trapmf_down(v, 0, 4) for v in x], color=RED,   lw=1.5, label='Rendah')
        ax.plot(x, [trimf(v, 2, 5, 8)    for v in x], color=GREEN, lw=1.5, label='Sedang')
        ax.plot(x, [trapmf_up(v, 6, 10)  for v in x], color=BLUE,  lw=1.5, label='Tinggi')
        ax.axvline(aktv, color='white', lw=1, linestyle='--', alpha=0.7)
        ax.scatter([aktv]*3, [mu_a['rendah'], mu_a['sedang'], mu_a['tinggi']],
                   color='white', zorder=5, s=18)
        style_ax(ax, f"Aktivitas  [{aktv:.0f}/10]")
        ax.legend(fontsize=6, labelcolor='#94a3b8', facecolor='#1e293b',
                  edgecolor='none', loc='upper right')

        # Output Risiko
        ax = axes[4]
        z_arr = np.linspace(0, 100, 400)
        ax.plot(z_arr, [trapmf_down(v, 0, 40)  for v in z_arr], color=GREEN,  lw=1.5, label='Rendah')
        ax.plot(z_arr, [trapmf_up(v, 30, 70)   for v in z_arr], color=ORANGE, lw=1.5, label='Sedang')
        ax.plot(z_arr, [trapmf_up(v, 60, 100)  for v in z_arr], color=RED,    lw=1.5, label='Tinggi')
        ax.axvline(z_result, color='white', lw=1.5, linestyle='--', alpha=0.9)
        ax.text(z_result + 1, 0.85, f"z={z_result:.1f}", color='white', fontsize=7)
        style_ax(ax, "Output Risiko")
        ax.legend(fontsize=6, labelcolor='#94a3b8', facecolor='#1e293b',
                  edgecolor='none', loc='upper left')

        canvas = FigureCanvasTkAgg(fig, master=self.tab_grafik)
        canvas.draw()
        canvas.get_tk_widget().pack(fill='both', expand=True)

    def _reset(self):
        defaults = {'umur': 45, 'bmi': 24, 'gula': 100, 'aktivitas': 5}
        for k, v in defaults.items():
            self.vars[k].set(v)
            self._update_entry(k)

        self.lbl_status.config(text="—", fg="#334155")
        self.lbl_skor.config(text="Masukkan data pasien lalu klik Hitung Risiko", fg="#64748b")
        self.lbl_info.config(text="")
        self.lbl_defuzz.config(text="")

        for item in self.tree.get_children():
            self.tree.delete(item)

        for w in self.tab_grafik.winfo_children():
            w.destroy()

        tk.Label(self.tab_grafik,
                 text="Klik 'Hitung Risiko' untuk melihat grafik",
                 font=("Segoe UI", 11), fg="#475569", bg="#0f172a").pack(expand=True)

    def _build_footer(self):
        ft = tk.Frame(self, bg="#1e293b", pady=6)
        ft.pack(fill='x', side='bottom')
        tk.Label(ft, text="Fuzzy Tsukamoto  |  Kelompok 7  |  2026",
                 font=("Segoe UI", 8), fg="#475569", bg="#1e293b").pack()


# MAIN

if __name__ == "__main__":
    app = FuzzyDiabetesApp()
    app.mainloop()
