import tkinter as tk
import sqlite3
from datetime import datetime

def init_db():
    conn = sqlite3.connect("hydrosentinel.db")
    c = conn.cursor()

    c.execute('''CREATE TABLE IF NOT EXISTS dams
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  name TEXT,
                  location TEXT,
                  river TEXT,
                  distance TEXT,
                  water_level TEXT,
                  status TEXT)''')

    
    c.execute("SELECT COUNT(*) FROM dams")
    if c.fetchone()[0] == 0:
        dams_data = [
            ("San Roque Dam", "San Manuel / San Nicolas, Pangasinan", "Agno River", "~45 km NE", "285.2 m", "MODERATE RISK"),
            ("Paitan Dam", "Santa Maria, Pangasinan", "Lower Agno River", "~35 km N", "45.5 m", "NORMAL"),
            ("Dumoloc Dam", "Bugallon, Pangasinan", "Dumoloc River", "~30 km SW", "22.3 m", "NORMAL"),
            ("Cayanga Dam", "Bugallon, Pangasinan", "Dumoloc River", "~30 km SW", "18.5 m", "NORMAL"),
            ("Cabatuan Dam", "Aguilar / Santa Cruz, Pangasinan", "Cabatuan River", "~38 km SW", "12.8 m", "NORMAL"),
            ("Binga Dam", "Itogon, Benguet", "Agno River (Upstream)", "~75 km NE", "575.2 m", "MONITORING"),
            ("Ambuklao Dam", "Bokod, Benguet", "Agno River (Upstream)", "~90 km NE", "751.8 m", "NORMAL"),
        ]
        c.executemany("INSERT INTO dams VALUES (NULL,?,?,?,?,?,?)", dams_data)

    conn.commit()
    conn.close()

class HydroSentinel:
    def __init__(self, root):
        self.root = root
        self.root.title("HYDRO SENTINEL - Dagupan City Flood Monitoring")
        self.root.geometry("1100x700")
        self.root.configure(bg="#eaf6ff")

        init_db()

        
        header = tk.Frame(root, bg="#0a4a7a", height=70)
        header.pack(fill="x")
        header.pack_propagate(False)
        tk.Label(header, text="HYDRO SENTINEL", font=("Arial Black", 22), bg="#0a4a7a", fg="white").pack(side="left", padx=20, pady=10)
        tk.Label(header, text="Dagupan City | Flood & Dam Early Warning System", font=("Arial", 11), bg="#0a4a7a", fg="#cfe8ff").pack(side="left")
        
        
        self.time_label = tk.Label(header, font=("Arial", 11, "bold"), bg="#0a4a7a", fg="white")
        self.time_label.pack(side="right", padx=20, pady=10)
        self.update_clock()

        
        main = tk.Frame(root, bg="#eaf6ff")
        main.pack(fill="both", expand=True)

        
        sidebar = tk.Frame(main, bg="#ffffff", width=250)
        sidebar.pack(side="left", fill="y", padx=10, pady=10)
        sidebar.pack_propagate(False)
        tk.Label(sidebar, text="MENU", font=("Arial", 12, "bold"), bg="white", fg="#0a4a7a").pack(pady=15)

        
        content_wrapper = tk.Frame(main, bg="white")
        content_wrapper.pack(side="right", fill="both", expand=True, padx=(0,10), pady=10)

        self.canvas = tk.Canvas(content_wrapper, bg="white", highlightthickness=0)
        self.scrollbar = tk.Scrollbar(content_wrapper, orient="vertical", command=self.canvas.yview)
        self.content_frame = tk.Frame(self.canvas, bg="white")

        self.content_frame.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        self.canvas.create_window((0, 0), window=self.content_frame, anchor="nw")
        self.canvas.configure(yscrollcommand=self.scrollbar.set)

        self.canvas.pack(side="left", fill="both", expand=True)
        self.scrollbar.pack(side="right", fill="y")
        self.canvas.bind_all("<MouseWheel>", lambda e: self.canvas.yview_scroll(int(-1*(e.delta/120)), "units"))

        menus = [
            ("DAM STATUS", self.show_dam),
            ("FLOOD AREAS", self.show_flood),
            ("HOTLINES", self.show_hotlines),
            ("SAFETY TIPS", self.show_tips),
            ("NEEDS", self.show_needs),
            ("NEARBY EVACUATION", self.show_evac),
        ]

        for text, cmd in menus:
            tk.Button(sidebar, text=text, font=("Arial", 11, "bold"), bg="#0a4a7a", fg="white",
                      activebackground="#1480d6", relief="flat", height=2, anchor="w", padx=15,
                      command=cmd, cursor="hand2").pack(fill="x", pady=5, padx=10)

        self.show_dam()

    def update_clock(self):
        now = datetime.now().strftime("%Y-%m-%d  %I:%M:%S %p")
        self.time_label.config(text=now)
        self.root.after(1000, self.update_clock)  # Update every 1 second

    def clear_content(self):
        for w in self.content_frame.winfo_children():
            w.destroy()

    def section_title(self, title, subtitle=""):
        tk.Label(self.content_frame, text=title, font=("Arial", 18, "bold"), bg="white", fg="#0a4a7a").pack(anchor="w", padx=20, pady=(20,5))
        if subtitle:
            tk.Label(self.content_frame, text=subtitle, font=("Arial", 11), bg="white", fg="gray").pack(anchor="w", padx=20)

    
    def show_dam(self):
        self.clear_content()
        self.section_title("Dams Around Dagupan & Pangasinan", "Data retrieved from database")

        conn = sqlite3.connect("hydrosentinel.db")
        c = conn.cursor()
        c.execute("SELECT name, location, river, distance, water_level, status FROM dams")
        dams = c.fetchall()
        conn.close()

        for name, loc, river, dist, level, status in dams:
            if "CRITICAL" in status or "HIGH" in status:
                color = "#e74c3c"
            elif "MODERATE" in status or "MONITORING" in status or "WARNING" in status:
                color = "#f39c12"
            else:
                color = "#2ecc71"

            card = tk.Frame(self.content_frame, bg="white", bd=1, relief="solid")
            card.pack(fill="x", padx=20, pady=8)

            tk.Label(card, text=name, font=("Arial", 13, "bold"), bg="white", fg="#0a4a7a").pack(anchor="w", padx=15, pady=(10,2))
            tk.Label(card, text="Location: " + loc, font=("Arial", 9), bg="white", fg="gray").pack(anchor="w", padx=15)
            tk.Label(card, text="River: " + river, font=("Arial", 9), bg="white").pack(anchor="w", padx=15)
            tk.Label(card, text="Distance from Dagupan: " + dist, font=("Arial", 9), bg="white").pack(anchor="w", padx=15)
            tk.Label(card, text="Water Level: " + level, font=("Arial", 10), bg="white").pack(anchor="w", padx=15, pady=(5,0))
            tk.Label(card, text="STATUS: " + status, font=("Arial", 11, "bold"), bg=color, fg="white", padx=10, pady=3).pack(anchor="w", padx=15, pady=(8,10))

    
    def show_flood(self):
        self.clear_content()
        self.section_title("Flood Affected Areas - Dagupan City", "Based on CDRRMO reports")
        groups = [
            ("HIGHLY FLOODED (Chest to waist)", ["Malued - Sitio Fisheries, Calarin", "Lasip Chico & Lasip Grande", "Bacayao Norte / Sur", "Pantal, Pogo Chico/Grande", "Mayombo, Tapuac"]),
            ("MODERATE FLOODED (Knee deep)", ["Barangay I, II, III, IV", "Herrero-Perez, Poblacion Oeste", "Caranglaan, Bolosan, Salisay"]),
            ("ISLAND BARANGAYS - Isolation Risk", ["Lomboy", "Salapingao", "Pugaro, Carael, Calmay"])
        ]
        for title, areas in groups:
            f = tk.Frame(self.content_frame, bg="white")
            f.pack(fill="x", padx=20, pady=10)
            tk.Label(f, text=title, font=("Arial", 11, "bold"), bg="white", fg="#0a4a7a").pack(anchor="w")
            for a in areas:
                tk.Label(f, text=" - " + a, font=("Arial", 10), bg="white").pack(anchor="w", padx=10)

    
    def show_hotlines(self):
        self.clear_content()
        self.section_title("Emergency Hotlines - Dagupan City", "Call first before posting on social media")
        hotlines = [
            ("CDRRMO Dagupan (Main Rescue)", "0968-444-9598 / (075) 540-0363 / 0932-389-1488"),
            ("Dagupan City Emergency 911", "16-911 / 911"),
            ("BFP Dagupan (Fire)", "(075) 522-2772 / 0917-184-2611"),
            ("PNP Dagupan Police", "(075) 523-7463 / (075) 515-3123"),
            ("Red Cross Pangasinan", "143 / (075) 522-4101"),
            ("City Health Office", "(075) 522-8203"),
        ]
        for name, num in hotlines:
            row = tk.Frame(self.content_frame, bg="#f7fbff", bd=1, relief="solid")
            row.pack(fill="x", padx=20, pady=5)
            tk.Label(row, text=name, font=("Arial", 10, "bold"), bg="#f7fbff", width=35, anchor="w").pack(side="left", padx=10, pady=10)
            tk.Label(row, text=num, font=("Arial", 11, "bold"), bg="#f7fbff", fg="#c0392b").pack(side="left")

    
    def show_tips(self):
        self.clear_content()
        self.section_title("Flood Safety Tips")
        tips = {
            "BEFORE FLOOD": ["Charge phones, powerbanks, flashlight", "Prepare Go-Bag: docs, meds, food, water", "Elevate appliances, know evac center", "Monitor San Roque Dam & PAGASA"],
            "DURING FLOOD": ["Turn off electricity & gas", "Do NOT walk/drive through floodwater", "Evacuate early when Barangay says so", "Go to 2nd floor/roof if trapped - call CDRRMO"],
            "AFTER FLOOD": ["Boil water before drinking", "Check for snakes & live wires", "Clean with disinfectant", "Report damages to Barangay"]
        }
        for k, v in tips.items():
            f = tk.Frame(self.content_frame, bg="white")
            f.pack(fill="x", padx=20, pady=8)
            tk.Label(f, text=k, font=("Arial", 11, "bold"), bg="#0a4a7a", fg="white", padx=10, pady=3).pack(anchor="w")
            for t in v:
                tk.Label(f, text=" - " + t, font=("Arial", 10), bg="white", anchor="w").pack(fill="x", padx=10, pady=2)

    
    def show_needs(self):
        self.clear_content()
        self.section_title("Essential Needs During Flood", "Priority needs for evacuees in Dagupan City")
        needs = [
            ("Food & Water", "Food Packs, Bigas, Canned Goods, Potable Drinking Water"),
            ("Hygiene Kits", "Sabon, Shampoo, Toothbrush, Napkin, Diaper, Alcohol, Sanitizer"),
            ("Health & Medicine", "Medicines, First Aid Kit, Maintenance meds for seniors"),
            ("Clothing & Shelter", "Clothing, Blankets, Banig, Kulambo, Trapo, Slippers"),
            ("Power & Light", "Flashlight, Batteries, Powerbank, Candles, Matches, Lighter"),
            ("Baby Needs", "Baby Milk, Bottles, Diapers, Baby Blanket, Wipes"),
            ("Rescue & Safety", "Rubber Boat, Life Vest, Whistle, Rope, Emergency Ladder"),
        ]
        for title, desc in needs:
            row = tk.Frame(self.content_frame, bg="#f7fbff", bd=1, relief="solid")
            row.pack(fill="x", padx=20, pady=6)
            tk.Label(row, text=title, font=("Arial", 11, "bold"), bg="#f7fbff", width=22, anchor="w").pack(side="left", padx=15, pady=12)
            tk.Label(row, text=desc, font=("Arial", 10), bg="#f7fbff", fg="#333", anchor="w").pack(side="left", pady=12)

    
    def show_evac(self):
        self.clear_content()
        self.section_title("Nearby Evacuation Centers - Dagupan City", "2-3 storey buildings accredited by CDRRMO")
        evac = [
            ("Dagupan City Astrodome - MAIN", "Poblacion - 100+ families", "MAIN"),
            ("Bonuan Boquig Evacuation Center", "Bonuan Boquig", "3-Storey"),
            ("Pugaro Evacuation Center", "Island Brgy. Pugaro", "Island"),
            ("Salapingao Evacuation Center", "Island Brgy. Salapingao", "Island"),
            ("West Central I & II Elementary", "Tapuac / Poblacion", "School"),
            ("North Central Elem - Bonuan Gueset", "Bonuan Gueset", "School"),
            ("Calmay Elementary Schools", "Calmay Island", "School"),
            ("Bacayao Sur/Norte Covered Court", "Bacayao", "Court"),
        ]
        for name, loc, tag in evac:
            row = tk.Frame(self.content_frame, bg="white", bd=1, relief="solid")
            row.pack(fill="x", padx=20, pady=5)
            left = tk.Frame(row, bg="white")
            left.pack(side="left", padx=10, pady=10)
            tk.Label(left, text=name, font=("Arial", 11, "bold"), bg="white", anchor="w").pack(anchor="w")
            tk.Label(left, text=loc, font=("Arial", 9), bg="white", fg="gray", anchor="w").pack(anchor="w")
            tk.Label(row, text=tag, font=("Arial", 8, "bold"), bg="#e1f0ff", fg="#0a4a7a", padx=8, pady=3).pack(side="right", padx=15)

if __name__=="__main__":
    root = tk.Tk()
    app = HydroSentinel(root)
    root.mainloop()
