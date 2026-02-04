# Ian-s-program-python-code
Testem o meu jogo baseado em D&D para eu ter mais ideias de como melhorar!!
import random

# ======================
# 🎲 UTILIDADES
# ======================
def rolar_dado(lados=20):
    return random.randint(1, lados)

# ======================
# 👤 PERSONAGEM
# ======================
def criar_personagem():
    nome = input("Nome do herói: ")

    print("1 - Guerreiro | 2 - Mago | 3 - Bardo")
    c = input("Classe: ")

    if c == "1":
        classe = "Guerreiro"
        vida = 55
        habilidades = ["Golpe Poderoso", "Postura Defensiva"]
    elif c == "2":
        classe = "Mago"
        vida = 35
        habilidades = ["Bola de Fogo", "Escudo Arcano"]
    else:
        classe = "Bardo"
        vida = 42
        habilidades = ["Canção Inspiradora", "Veneno Melódico"]

    return {
        "nome": nome,
        "classe": classe,
        "nivel": 1,
        "xp": 0,
        "xp_prox": 100,
        "vida": vida,
        "vida_max": vida,
        "inventario": ["Poção de Cura"],
        "habilidades": habilidades,
        "escudo_turnos": 0,
        "postura_turnos": 0,
        "cooldown_golpe": 0
    }

# ======================
# 🎒 ITENS
# ======================
def usar_item(j):
    if not j["inventario"]:
        print("🎒 Inventário vazio!")
        return
    j["inventario"].pop()
    cura = 12 + rolar_dado(20)
    j["vida"] = min(j["vida"] + cura, j["vida_max"])
    print(f"🧪 Você recupera {cura} HP!")

# ======================
# 🧙 HABILIDADES
# ======================
def habilidade(j, i):
    for idx, h in enumerate(j["habilidades"]):
        print(f"{idx+1} - {h}")
    h = int(input("> ")) - 1
    nome = j["habilidades"][h]

    if nome == "Golpe Poderoso":
        if j["cooldown_golpe"] > 0:
            print("⏳ Golpe em recarga!")
            return
        dano = 30 + rolar_dado(30)
        i["vida"] -= dano
        j["cooldown_golpe"] = 4
        print(f"💥 Golpe Poderoso causa {dano}!")

    elif nome == "Postura Defensiva":
        j["postura_turnos"] = 5
        print("🛡️ Postura Defensiva ativada (counter pronta)")

    elif nome == "Bola de Fogo":
        dano = rolar_dado(12) + 10
        i["vida"] -= dano
        print(f"🔥 Bola de Fogo causa {dano}")

    elif nome == "Escudo Arcano":
        j["escudo_turnos"] = 3
        print("🔮 Escudo Arcano por 3 turnos")

    elif nome == "Canção Inspiradora":
        cura = 15 + rolar_dado(20)
        j["vida"] = min(j["vida"] + cura, j["vida_max"])
        print(f"🎵 Cura {cura}")

    elif nome == "Veneno Melódico":
        i["veneno"] = 5
        print("☠️ Inimigo envenenado!")

# ======================
# ⚔️ COMBATE
# ======================
def combate(j, i):
    print(f"\n⚔️ {i['nome']} aparece!")

    while j["vida"] > 0 and i["vida"] > 0:
        print(f"❤️ {j['vida']} | 👹 {i['vida']}")
        print("1 Atacar | 2 Habilidade | 3 Item")
        a = input("> ")

        if j["cooldown_golpe"] > 0:
            j["cooldown_golpe"] -= 1

        if a == "1":
            dano = 20 + rolar_dado(20)
            i["vida"] -= dano
            print(f"⚔️ Você causa {dano}")
        elif a == "2":
            habilidade(j, i)
        elif a == "3":
            usar_item(j)

        if i["vida"] <= 0:
            break

        # Veneno
        if i.get("veneno", 0) > 0:
            i["vida"] -= 5
            i["veneno"] -= 1
            print("☠️ Veneno causa 5")

        # Ataque inimigo
        dano = i["ataque"]

        if j["escudo_turnos"] > 0:
            j["escudo_turnos"] -= 1
            print("🔮 Escudo bloqueia tudo")
            continue

        if j["postura_turnos"] > 0:
            j["postura_turnos"] -= 1
            if dano >= 15:
                refletido = int(dano * 0.25)
                i["vida"] -= refletido
                print(f"🛡️ COUNTER! Dano anulado e {refletido} refletido!")
            else:
                dano = int(dano * 0.4)
                j["vida"] -= dano
                print(f"🛡️ Postura reduz dano para {dano}")
        else:
            j["vida"] -= dano
            print(f"👹 {dano} de dano")

    if j["vida"] > 0:
        ganho = i["xp"]
        j["xp"] += ganho
        print(f"✨ Ganhou {ganho} XP!")

        if random.random() < 0.6:
            j["inventario"].append("Poção de Cura")
            print("🎁 Encontrou uma Poção!")

        return True
    else:
        print("💀 Você caiu em batalha...")
        return False

# ======================
# 🗺️ MAPA & ENCONTROS
# ======================
MAPA = [
    ["P",".",".",".","."],
    [".",".",".",".","."],
    [".",".",".",".","."],
    [".",".",".",".","."],
    [".",".",".",".","V"]
]

pos = [0,0]

def mostrar_mapa():
    for l in MAPA:
        print(" ".join(l))

def encontro_aleatorio():
    r = random.random()
    if r < 0.6:
        return {"nome":"Aberração","vida":30,"ataque":8,"xp":40,"veneno":0}
    elif r < 0.85:
        return {"nome":"Aberração Mutante","vida":40,"ataque":10,"xp":60,"veneno":0}
    else:
        return {"nome":"DEMODEMON – DEMOGORGON","vida":95,"ataque":14,"xp":120,"veneno":0}

# ======================
# 🎮 JOGO
# ======================
j = criar_personagem()

while True:
    mostrar_mapa()
    m = input("Mover (w/a/s/d): ")

    nx, ny = pos
    if m == "w": nx -= 1
    if m == "s": nx += 1
    if m == "a": ny -= 1
    if m == "d": ny += 1

    if 0 <= nx < 5 and 0 <= ny < 5:
        pos = [nx, ny]
        print("🚶 Você se move...")

        # Encontros MUITO frequentes
        if random.random() < 0.7:
            inimigo = encontro_aleatorio()
            if not combate(j, inimigo):
                break

        # Vecna fixo
        if pos == [4,4]:
            boss = {"nome":"VECNA – O SUPREMO","vida":220,"ataque":18,"xp":300,"veneno":0}
            if combate(j, boss):
                print("\n🌌 Você encara o vazio… e vence.")
                print("🏆 FIM DA JORNADA")
            break
