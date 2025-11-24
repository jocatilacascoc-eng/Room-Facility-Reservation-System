# Room-Facility-Reservation-System
class User:
    def __init__(self,name,role): self.name,self.role=name,role

class Room:
    def __init__(self,n,t):
        self.n,self.t=n,t
        self.reserved=False; self.req=None; self.by=None
        self.day=None; self.pref=None

    def request(self,u):
        if self.reserved: print(f"Room {self.n} already reserved by {self.by}."); return
        if self.req: print(f"Room {self.n} already requested by {self.req}."); return
        while True:
            try:
                d=int(input("Enter preferred day (1-31): "))
                if 1<=d<=31: self.pref=d; break
                else: print("Invalid day. Try again.")
            except: print("Invalid input. Try again.")
        self.req=u.name; print(f"Requested Room {self.n} (Preferred Day: {self.pref}).")

    def cancel(self,u):
        if self.reserved:
            if self.by==u.name or u.role=="Admin":
                self.reserved=self.by=self.day=self.pref=None
                print(f"Reservation for Room {self.n} cancelled.")
            else: print("You cannot cancel this reservation.")
        elif self.req:
            if self.req==u.name or u.role=="Admin":
                self.req=self.pref=None; print(f"Request for Room {self.n} cancelled.")
            else: print("You cannot cancel this request.")
        else: print("Room has no request/reservation.")

    def __str__(self):
        if self.reserved: s=f"Reserved by {self.by} (Day {self.day})"
        elif self.req: s=f"Requested by {self.req} (Preferred Day{self.pref})"
        else: s="Available"
        return f"Room {self.n} ({self.t}) - {s}"

class System:
    rooms=[Room(101,"Single"),Room(102,"Single"),Room(201,"Double"),Room(202,"Conference")]
    def __init__(self,u): self.u=u
    def get(self,n): return next((r for r in self.rooms if r.n==n),None)
    def show(self): [print(r) for r in self.rooms]
    def inp(self,msg):
        try:
            s=input(msg).strip()
            return list({int(x) for x in s.split(",")}) if s else []
        except: print("Invalid input."); return []

    def request(self):
        for n in self.inp("Enter room numbers to request: "):
            r=self.get(n); r.request(self.u) if r else print("Invalid room.")

    def cancel(self):
        if not any(r.req==self.u.name or r.by==self.u.name for r in self.rooms):
            print("You have no rooms to cancel."); return
        for n in self.inp("Enter room numbers to cancel: "):
            r=self.get(n); r.cancel(self.u) if r else print("Invalid room.")

    def manage(self):
        rs=[r for r in self.rooms if r.reserved]
        if not rs: print("No reservations."); return
        [print(f"Room {r.n} - {r.by} (Day {r.day}, Pref {r.pref})") for r in rs]
        try:
            r=self.get(int(input("Enter room number: ")))
            if not r or not r.reserved: print("Invalid."); return
            d=int(input("Enter approved day (1-31): "))
            if 1<=d<=31:
                r.day=d; r.pref=None
                print("Approved day set. Preferred date cleared.")
            else: print("Invalid day.")
        except: print("Invalid day.")

    def approve(self):
        p=[r for r in self.rooms if r.req]
        if not p: print("No pending requests."); return
        [print(f"Room {r.n} requested by {r.req} (Preferred {r.pref})") for r in p]
        act=input("Approve or Reject? ").lower()
        if act not in("approve","reject"): print("Invalid."); return
        for n in self.inp("Enter rooms to process: "):
            r=self.get(n)
            if not r or not r.req: print("Invalid room."); continue
            if act=="approve":
                r.by=r.req; r.req=None; r.reserved=True; print(f"Room {r.n} approved.")
            else:
                r.req=r.pref=None; print(f"Room {r.n} rejected.")

    def menu(self):
        while True:
            print("\n--- ROOM STATUS ---"); self.show()
            if self.u.role=="Admin":
                c=input("\n1. Approve/Reject\n2. Manage Schedule\n3. Logout\nChoose: ")
                if c=="1": self.approve()
                elif c=="2": self.manage()
                elif c=="3": break
                else: print("Invalid option.")
            else:
                c=input("\n1. Request Room\n2. Cancel Room\n3. Logout\nChoose: ")
                if c=="1": self.request()
                elif c=="2": self.cancel()
                elif c=="3": break
                else: print("Invalid option.")

def login():
    name=input("\nEnter your name: ") or "User"
    while True:
        c=input("Select role:\n1. Faculty/Student\n2. Admin\n3. Exit\nChoose: ")
        if c=="1": return User(name,"Faculty/Student")
        if c=="2": return User(name,"Admin")
        if c=="3": exit()
        print("Invalid.")

if __name__=="__main__":
    while True: System(login()).menu()
