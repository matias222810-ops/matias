# app.py
import streamlit as st
import random
import difflib
import time

# ===================== CONFIG & THEME =====================
st.set_page_config(page_title="Trivia Game Ultra", page_icon="🎮", layout="wide")

PRIMARY = "#22c55e"     # green
ACCENT = "#38bdf8"      # sky
WARNING = "#f59e0b"     # amber
DANGER = "#ef4444"      # red
CARD_BG = "#0f172a"     # slate-900
TEXT = "#e5e7eb"        # gray-200
MUTED = "#94a3b8"       # slate-400

CUSTOM_CSS = f"""
<style>
    .main .block-container {{
        padding-top: 1rem;
        padding-bottom: 2rem;
        max-width: 1100px;
    }}
    .stButton>button {{
        border-radius: 12px;
        padding: .7rem 1rem;
        font-weight: 700;
    }}
    .btn-green>button {{ background:{PRIMARY} !important; color:#0b1220 !important; border:0 }}
    .btn-sky>button   {{ background:{ACCENT} !important; color:#0b1220 !important; border:0 }}
    .btn-amber>button {{ background:{WARNING} !important; color:#0b1220 !important; border:0 }}
    .btn-red>button   {{ background:{DANGER} !important; color:white !important; border:0 }}
    .card {{
        background: {CARD_BG};
        border: 1px solid #1f2937;
        border-radius: 16px; padding: 18px;
    }}
    .title {{
        font-size: 2rem; font-weight:800; color:{TEXT};
    }}
    .subtitle {{
        font-size: 1.1rem; color:{MUTED};
    }}
    .question {{
        font-size: 1.25rem; font-weight:700; color:{TEXT};
    }}
    .timer-wrap {{
        background: #1f2937; border-radius: 12px; height: 16px; width:100%;
        overflow: hidden; border: 1px solid #334155;
    }}
    .timer-bar {{
        height: 100%; background: {PRIMARY}; transition: width .3s linear;
    }}
</style>
"""
st.markdown(CUSTOM_CSS, unsafe_allow_html=True)

# ===================== DATA =====================
def build_questions():
    return {
        "Fútbol": {
            1: [
                {"pregunta": "¿Quién ganó el Mundial de Fútbol 2018?", "opciones": ["Francia", "Croacia", "Brasil", "Alemania"], "respuesta": "Francia", "pista": "El equipo de Kylian Mbappé."},
                {"pregunta": "¿Qué jugador es conocido como 'La Pulga'?", "opciones": ["Messi", "Neymar", "Ronaldo", "Suárez"], "respuesta": "Messi", "pista": "Juega para PSG."},
                {"pregunta": "¿En qué país se originó el fútbol?", "opciones": ["Inglaterra", "Brasil", "España", "Italia"], "respuesta": "Inglaterra", "pista": "País de la Premier League."},
                {"pregunta": "¿Cuántos jugadores tiene un equipo en cancha?", "opciones": ["11", "10", "9", "12"], "respuesta": "11", "pista": "Número estándar."},
                {"pregunta": "¿Cuál es el máximo goleador de la Champions League?", "opciones": ["Cristiano Ronaldo", "Messi", "Lewandowski", "Raúl"], "respuesta": "Cristiano Ronaldo", "pista": "Jugador portugués."},
                {"pregunta": "¿Qué significa 'offside' en fútbol?", "opciones": ["Fuera de juego", "Tarjeta amarilla", "Falta", "Gol anulado"], "respuesta": "Fuera de juego", "pista": "Regla polémica."},
                {"pregunta": "¿Cuál es el equipo más ganador de la Copa América?", "opciones": ["Uruguay", "Argentina", "Brasil", "Chile"], "respuesta": "Uruguay", "pista": "País charrúa."},
                {"pregunta": "¿Qué país organizó el Mundial de 2014?", "opciones": ["Brasil", "Rusia", "Sudáfrica", "Alemania"], "respuesta": "Brasil", "pista": "País sudamericano."},
                {"pregunta": "¿Quién fue el 'Rey del Fútbol'?", "opciones": ["Pelé", "Maradona", "Zidane", "Messi"], "respuesta": "Pelé", "pista": "Jugó en Santos y Brasil."},
                {"pregunta": "¿En qué posición juega un portero?", "opciones": ["Defensa", "Delantero", "Guardameta", "Mediocampista"], "respuesta": "Guardameta", "pista": "Protege el arco."},
            ],
            2: [
                {"pregunta": "¿Cuántos minutos dura un partido oficial de fútbol?", "opciones": ["90", "60", "120", "80"], "respuesta": "90", "pista": "Dos tiempos de 45 minutos."},
                {"pregunta": "¿Qué país ganó la Eurocopa 2020?", "opciones": ["Italia", "Inglaterra", "España", "Francia"], "respuesta": "Italia", "pista": "Equipo azzurro."},
                {"pregunta": "¿Quién fue el máximo goleador del Mundial 2014?", "opciones": ["James Rodríguez", "Neymar", "Müller", "Ronaldo"], "respuesta": "James Rodríguez", "pista": "Jugador colombiano."},
                {"pregunta": "¿Qué equipo tiene el récord de más Champions League ganadas?", "opciones": ["Real Madrid", "AC Milan", "Liverpool", "Bayern Munich"], "respuesta": "Real Madrid", "pista": "Club blanco."},
                {"pregunta": "¿Cuál es el nombre del torneo sudamericano de clubes?", "opciones": ["Copa Libertadores", "Copa Sudamericana", "Copa América", "Supercopa"], "respuesta": "Copa Libertadores", "pista": "Similar a la Champions."},
                {"pregunta": "¿En qué año debutó Lionel Messi con la selección Argentina?", "opciones": ["2005", "2003", "2007", "2004"], "respuesta": "2005", "pista": "Debutó con 18 años."},
                {"pregunta": "¿Qué equipo argentino es conocido como 'Los Millonarios'?", "opciones": ["River Plate", "Boca Juniors", "Racing Club", "Independiente"], "respuesta": "River Plate", "pista": "Rival de Boca."},
                {"pregunta": "¿Qué significa VAR en fútbol?", "opciones": ["Video Assistant Referee", "Video Analysis Replay", "Virtual Assistant Referee", "Video Automated Review"], "respuesta": "Video Assistant Referee", "pista": "Ayuda arbitral."},
                {"pregunta": "¿Cuántos goles marcó Diego Maradona en el Mundial 1986?", "opciones": ["5", "4", "6", "3"], "respuesta": "5", "pista": "Incluye el 'Gol del Siglo'."},
                {"pregunta": "¿Qué significa el término 'Hat-trick'?", "opciones": ["Tres goles en un partido", "Tres asistencias", "Tres tarjetas amarillas", "Tres penales fallados"], "respuesta": "Tres goles en un partido", "pista": "Logro goleador."},
            ],
        },
        "Cultura General": {
            1: [
                {"pregunta": "¿Cuál es la capital de Francia?", "opciones": ["París", "Londres", "Berlín", "Madrid"], "respuesta": "París", "pista": "Ciudad de la Torre Eiffel."},
                {"pregunta": "¿Quién pintó la Mona Lisa?", "opciones": ["Leonardo da Vinci", "Pablo Picasso", "Vincent Van Gogh", "Michelangelo"], "respuesta": "Leonardo da Vinci", "pista": "Artista renacentista."},
                {"pregunta": "¿Cuál es el río más largo del mundo?", "opciones": ["Amazonas", "Nilo", "Yangtsé", "Misisipi"], "respuesta": "Amazonas", "pista": "Está en Sudamérica."},
                {"pregunta": "¿En qué continente está Egipto?", "opciones": ["África", "Asia", "Europa", "Oceanía"], "respuesta": "África", "pista": "Famoso por las pirámides."},
                {"pregunta": "¿Cuál es el idioma oficial de Brasil?", "opciones": ["Portugués", "Español", "Inglés", "Francés"], "respuesta": "Portugués", "pista": "No es español."},
                {"pregunta": "¿Quién escribió 'Cien años de soledad'?", "opciones": ["Gabriel García Márquez", "Mario Vargas Llosa", "Julio Cortázar", "Isabel Allende"], "respuesta": "Gabriel García Márquez", "pista": "Realismo mágico."},
                {"pregunta": "¿En qué país se encuentra la Torre de Pisa?", "opciones": ["Italia", "Francia", "España", "Alemania"], "respuesta": "Italia", "pista": "Europa."},
                {"pregunta": "¿Cuál es el planeta más cercano al Sol?", "opciones": ["Mercurio", "Venus", "Tierra", "Marte"], "respuesta": "Mercurio", "pista": "Pequeño y rápido."},
                {"pregunta": "¿Quién fue el primer hombre en pisar la Luna?", "opciones": ["Neil Armstrong", "Buzz Aldrin", "Yuri Gagarin", "Michael Collins"], "respuesta": "Neil Armstrong", "pista": "1969."},
                {"pregunta": "¿Cuál es el símbolo químico del oro?", "opciones": ["Au", "Ag", "O", "Fe"], "respuesta": "Au", "pista": "Elemento precioso."},
            ],
            2: [
                {"pregunta": "¿Cuál es la lengua más hablada del mundo?", "opciones": ["Chino mandarín", "Inglés", "Español", "Hindi"], "respuesta": "Chino mandarín", "pista": "Más de mil millones de hablantes."},
                {"pregunta": "¿Quién escribió 'Don Quijote de la Mancha'?", "opciones": ["Miguel de Cervantes", "Lope de Vega", "Federico García Lorca", "Gabriel García Márquez"], "respuesta": "Miguel de Cervantes", "pista": "Literatura española."},
                {"pregunta": "¿En qué año cayó el Muro de Berlín?", "opciones": ["1989", "1991", "1980", "1975"], "respuesta": "1989", "pista": "Fin de la Guerra Fría."},
                {"pregunta": "¿Qué evento marcó el inicio de la Segunda Guerra Mundial?", "opciones": ["Invasión a Polonia", "Ataque a Pearl Harbor", "Caída de Berlín", "Batalla de Stalingrado"], "respuesta": "Invasión a Polonia", "pista": "1939."},
                {"pregunta": "¿Quién fue Marie Curie?", "opciones": ["Científica pionera en radioactividad", "Pintora famosa", "Escritora francesa", "Exploradora"], "respuesta": "Científica pionera en radioactividad", "pista": "Dos premios Nobel."},
                {"pregunta": "¿Cuál es la capital de Australia?", "opciones": ["Canberra", "Sídney", "Melbourne", "Brisbane"], "respuesta": "Canberra", "pista": "No es Sídney."},
                {"pregunta": "¿Quién fue Nelson Mandela?", "opciones": ["Líder sudafricano contra el apartheid", "Explorador africano", "Músico famoso", "Político brasileño"], "respuesta": "Líder sudafricano contra el apartheid", "pista": "Presidente de Sudáfrica."},
                {"pregunta": "¿Qué país tiene la mayor cantidad de pirámides?", "opciones": ["Sudán", "Egipto", "México", "Perú"], "respuesta": "Sudán", "pista": "Más que Egipto."},
                {"pregunta": "¿Cuál es el océano más grande del mundo?", "opciones": ["Pacífico", "Atlántico", "Índico", "Ártico"], "respuesta": "Pacífico", "pista": "Entre América y Asia."},
                {"pregunta": "¿Qué artista es famoso por los 'Campbell's Soup'?", "opciones": ["Andy Warhol", "Salvador Dalí", "Pablo Picasso", "Claude Monet"], "respuesta": "Andy Warhol", "pista": "Pop art."},
            ],
        },
        "Matemáticas": {
            1: [
                {"pregunta": "¿Cuánto es 5 + 7?", "opciones": ["12", "10", "14", "11"], "respuesta": "12", "pista": "Suma básica."},
                {"pregunta": "¿Qué es un número primo?", "opciones": ["Divisible solo por 1 y sí mismo", "Número par", "Número impar", "Divisible por 2"], "respuesta": "Divisible solo por 1 y sí mismo", "pista": "Ejemplo: 2, 3, 5."},
                {"pregunta": "¿Cuánto es 9 x 9?", "opciones": ["81", "72", "99", "90"], "respuesta": "81", "pista": "Tabla del 9."},
                {"pregunta": "¿Cuál es la raíz cuadrada de 64?", "opciones": ["8", "6", "7", "9"], "respuesta": "8", "pista": "Número al cuadrado."},
                {"pregunta": "¿Qué valor tiene Pi (aproximadamente)?", "opciones": ["3.14", "2.14", "3.41", "4.13"], "respuesta": "3.14", "pista": "Número irracional."},
                {"pregunta": "¿Cuánto es 15 - 6?", "opciones": ["9", "8", "10", "7"], "respuesta": "9", "pista": "Resta básica."},
                {"pregunta": "¿Cuánto es 100 dividido entre 4?", "opciones": ["25", "20", "30", "15"], "respuesta": "25", "pista": "División sencilla."},
                {"pregunta": "¿Cuál es el número siguiente en la serie 2, 4, 6, 8...?", "opciones": ["10", "12", "9", "11"], "respuesta": "10", "pista": "Patrón de pares."},
                {"pregunta": "¿Qué es un ángulo recto?", "opciones": ["90 grados", "45 grados", "180 grados", "60 grados"], "respuesta": "90 grados", "pista": "Ángulo en forma de L."},
                {"pregunta": "¿Cuánto es 7 x 8?", "opciones": ["56", "54", "48", "58"], "respuesta": "56", "pista": "Tabla del 7."},
            ],
            2: [
                {"pregunta": "¿Cuánto es la suma de los ángulos internos de un triángulo?", "opciones": ["180 grados", "360 grados", "90 grados", "270 grados"], "respuesta": "180 grados", "pista": "Propiedad básica."},
                {"pregunta": "¿Cuál es el número primo más pequeño?", "opciones": ["2", "1", "3", "0"], "respuesta": "2", "pista": "Único par primo."},
                {"pregunta": "¿Qué representa el símbolo ∑?", "opciones": ["Sumatoria", "Producto", "Igualdad", "Diferencia"], "respuesta": "Sumatoria", "pista": "Matemáticas avanzadas."},
                {"pregunta": "¿Cuál es el valor de e (aproximado)?", "opciones": ["2.71", "3.14", "1.41", "2.18"], "respuesta": "2.71", "pista": "Número de Euler."},
                {"pregunta": "¿Qué es un número irracional?", "opciones": ["No puede expresarse como fracción", "Número entero", "Número negativo", "Número decimal exacto"], "respuesta": "No puede expresarse como fracción", "pista": "Ejemplo: Pi."},
                {"pregunta": "¿Cuál es el resultado de (3^3)?", "opciones": ["27", "9", "81", "18"], "respuesta": "27", "pista": "Potencia."},
                {"pregunta": "¿Cuál es el perímetro de un cuadrado de lado 5?", "opciones": ["20", "25", "10", "15"], "respuesta": "20", "pista": "4 lados iguales."},
                {"pregunta": "¿Cuánto es 1/2 + 1/4?", "opciones": ["3/4", "1/2", "1/4", "2/4"], "respuesta": "3/4", "pista": "Suma de fracciones."},
                {"pregunta": "¿Cuál es la fórmula para el área de un círculo?", "opciones": ["πr²", "2πr", "πd", "r²"], "respuesta": "πr²", "pista": "Área = pi por radio al cuadrado."},
                {"pregunta": "¿Cuál es el valor de la raíz cuadrada de 121?", "opciones": ["11", "10", "12", "9"], "respuesta": "11", "pista": "Número al cuadrado."},
            ]
        }
    }

CATS = build_questions()

# ===================== STATE HELPERS =====================
def reset_trivia(categoria=None, dificultad="Medio"):
    st.session_state.mode = "MENU" if categoria is None else "TRIVIA"
    st.session_state.categoria = categoria
    st.session_state.nivel = 1
    st.session_state.vidas = {"Fácil":5, "Medio":3, "Difícil":2}[dificultad]
    st.session_state.tiempo_por_pregunta = {"Fácil":25, "Medio":20, "Difícil":12}[dificultad]
    st.session_state.mult = {"Fácil":1.0, "Medio":1.2, "Difícil":1.5}[dificultad]
    st.session_state.puntaje = 0
    st.session_state.preguntas = random.sample(CATS[categoria][1], k=len(CATS[categoria][1]))
    st.session_state.q_idx = 0
    st.session_state.tiempo_restante = st.session_state.tiempo_por_pregunta
    st.session_state.last_tick = 0
    st.session_state.selected = None
    st.session_state.show_hint = False

def next_level_or_finish():
    cat = st.session_state.categoria
    lvl = st.session_state.nivel
    if lvl < max(CATS[cat].keys()):
        st.session_state.nivel += 1
        pool = CATS[cat][st.session_state.nivel][:]
        random.shuffle(pool)
        st.session_state.preguntas = pool
        st.session_state.q_idx = 0
        st.session_state.tiempo_restante = st.session_state.tiempo_por_pregunta
        st.success(f"¡Nivel {lvl} completado! Pasas a nivel {st.session_state.nivel}.")
    else:
        st.session_state.mode = "RESULT"

def lose_life(msg):
    st.session_state.vidas -= 1
    st.warning(msg)
    if st.session_state.vidas <= 0:
        st.session_state.mode = "RESULT"
    else:
        st.session_state.q_idx += 1
        if st.session_state.q_idx >= len(st.session_state.preguntas):
            next_level_or_finish()
        st.session_state.tiempo_restante = st.session_state.tiempo_por_pregunta
        st.session_state.selected = None
        st.session_state.show_hint = False

def verify_answer():
    sel = st.session_state.selected
    if sel is None:
        st.info("Elige una opción antes de confirmar.")
        return
    q = st.session_state.preguntas[st.session_state.q_idx]
    correcta = q["respuesta"]
    ratio = difflib.SequenceMatcher(None, sel.lower(), correcta.lower()).ratio()
    if sel == correcta or ratio > 0.7:
        bonus = int(10 * st.session_state.mult)
        bonus_t = int(st.session_state.tiempo_restante * 0.5)
        st.session_state.puntaje += bonus + bonus_t
        st.success(f"¡Correcto! +{bonus} y +{bonus_t} por tiempo. Puntaje: {st.session_state.puntaje}")
        st.session_state.q_idx += 1
        if st.session_state.q_idx >= len(st.session_state.preguntas):
            next_level_or_finish()
        st.session_state.tiempo_restante = st.session_state.tiempo_por_pregunta
        st.session_state.selected = None
        st.session_state.show_hint = False
    else:
        lose_life(f"Incorrecto. La respuesta era: {correcta}")

# ===================== INIT SESSION =====================
defaults = dict(
    mode="MENU", categoria=None, nivel=1, vidas=3, puntaje=0,
    preguntas=[], q_idx=0, tiempo_por_pregunta=20, tiempo_restante=20,
    last_tick=0, selected=None, show_hint=False,
    pen_intentos=5, pen_goles=0, pen_cpu=0, pen_sudden=False
)
for k, v in defaults.items():
    st.session_state.setdefault(k, v)

# ===================== SIDEBAR =====================
with st.sidebar:
    st.markdown(f"<div class='title'>🎮 Trivia Game Ultra</div>", unsafe_allow_html=True)
    st.write("")
    if st.session_state.mode in ("TRIVIA", "RESULT"):
        st.markdown(f"**Categoría:** {st.session_state.categoria or '—'}")
        st.markdown(f"**Nivel:** {st.session_state.nivel}")
        st.markdown(f"**Vidas:** {'❤️'*st.session_state.vidas if st.session_state.vidas>0 else '—'}")
        st.markdown(f"**Puntaje:** {st.session_state.puntaje}")
        st.divider()
        if st.button("🏠 Ir al menú", use_container_width=True):
            st.session_state.mode = "MENU"

# ===================== MENU =====================
def view_menu():
    st.markdown(f"<div class='title'>🧠 Elige un modo</div>", unsafe_allow_html=True)
    st.markdown(f"<div class='subtitle'>Selecciona una categoría para jugar, o prueba el extra de penaltis ⚽</div>", unsafe_allow_html=True)
    st.write("")
    c1, c2, c3 = st.columns(3)
    for i, cat in enumerate(CATS.keys()):
        with (c1 if i==0 else c2 if i==1 else c3):
            st.markdown(f"<div class='card'><div class='question'>📚 {cat}</div><div class='subtitle'>Dos niveles</div></div>", unsafe_allow_html=True)
            diff = st.segmented_control(
                f"Dificultad · {cat}", options=["Fácil", "Medio", "Difícil"],
                default="Medio", key=f"diff_{cat}"
            )
            st.write("")
            if st.container():
                k = f"btn_{cat}"
                _ = st.button(f"Jugar {cat}", key=k, use_container_width=True, type="primary")
                if _:
                    reset_trivia(cat, dificultad=diff)
                    st.rerun()

    st.write("")
    st.divider()
    st.markdown("<div class='title'>⚽ Extra: Penaltis</div>", unsafe_allow_html=True)
    cols = st.columns([1,1,1])
    with cols[1]:
        if st.container():
            btn = st.button("Jugar Penaltis", use_container_width=True)
            if btn:
                st.session_state.mode = "PENALTIS"
                st.session_state.pen_intentos = 5
                st.session_state.pen_goles = 0
                st.session_state.pen_cpu = 0
                st.session_state.pen_sudden = False
                st.rerun()

# ===================== TRIVIA VIEW (with timer) =====================
def timer_bar():
    total = st.session_state.tiempo_por_pregunta
    left = max(0, st.session_state.tiempo_restante)
    pct = int((left/total)*100) if total else 0
    color = PRIMARY if left > total/2 else (WARNING if left > total/4 else DANGER)
    st.markdown(
        f"<div class='timer-wrap'>"
        f"<div class='timer-bar' style='width:{pct}%; background:{color}'></div>"
        f"</div>",
        unsafe_allow_html=True
    )

def view_trivia():
    # auto-refresh each second, and decrement once per tick
    tick = st.experimental_data_editor if False else None  # (placeholder to avoid linter issues)
    tick_count = st.experimental_rerun if False else None   # (placeholder)

    tick = st.autorefresh if hasattr(st, "autorefresh") else None  # compatibility shim
    # Use st_autorefresh from streamlit >= 1.37
    try:
        tick_val = st.experimental_rerun  # just to check attribute exists
        # if we got here, ignore; fallback below
    except Exception:
        pass
    tick_id = st.experimental_get_query_params()  # no-op

    # Proper API:
    from streamlit.runtime.scriptrunner import add_script_run_ctx  # noqa: F401
    # We actually use built-in helper:
    tick = st.experimental_memo if False else None  # noop

def view_trivia_real():
    # Real timer using st_autorefresh
    tick = st.autorefresh(interval=1000, key="tick", limit=None)

    if st.session_state.last_tick != tick and st.session_state.mode == "TRIVIA":
        # Decrement 1s per tick
        st.session_state.tiempo_restante -= 1
        st.session_state.last_tick = tick
        if st.session_state.tiempo_restante <= 0:
            lose_life("¡Se acabó el tiempo!")
            st.rerun()

    q = st.session_state.preguntas[st.session_state.q_idx]
    st.markdown(f"<div class='card'>", unsafe_allow_html=True)
    st.markdown(f"<div class='question'>{q['pregunta']}</div>", unsafe_allow_html=True)
    st.write("")

    timer_bar()
    st.caption(f"Tiempo restante: {st.session_state.tiempo_restante} s")

    st.write("")
    # Opciones (barajadas determinísticamente por índice de pregunta)
    opciones = q["opciones"][:]
    random.Random(st.session_state.q_idx).shuffle(opciones)

    st.session_state.selected = st.radio(
        "Elige una opción", opciones,
        index=None if st.session_state.selected is None else opciones.index(st.session_state.selected),
        label_visibility="collapsed"
    )

    cols = st.columns([1,1,1])
    with cols[0]:
        st.button("💡 Pista", use_container_width=True, key=f"hint_{st.session_state.q_idx}",
                  on_click=lambda: st.session_state.update(show_hint=True))
    with cols[1]:
        st.button("✅ Confirmar", use_container_width=True, type="primary", on_click=verify_answer)
    with cols[2]:
        if st.button("🏠 Menú", use_container_width=True):
            st.session_state.mode = "MENU"
            st.rerun()

    if st.session_state.show_hint:
        st.info(q["pista"])

    st.markdown("</div>", unsafe_allow_html=True)

# ===================== RESULT VIEW =====================
def view_result():
    st.markdown(f"<div class='title'>🏁 Resultado</div>", unsafe_allow_html=True)
    st.markdown(f"<div class='card'><h3 style='color:{TEXT};margin:0'>Puntaje: {st.session_state.puntaje}</h3>", unsafe_allow_html=True)
    stars = "⭐" * (1 + min(4, st.session_state.puntaje // 60)) if st.session_state.puntaje > 0 else "✦"
    st.markdown(f"<div class='subtitle'>Nivel alcanzado: {st.session_state.nivel}</div>", unsafe_allow_html=True)
    st.markdown(f"<div style='font-size:1.6rem'> {stars} </div></div>", unsafe_allow_html=True)
    c1, c2 = st.columns(2)
    with c1:
        st.button("🔁 Volver a jugar (misma categoría)", use_container_width=True, on_click=lambda: reset_trivia(st.session_state.categoria))
    with c2:
        if st.button("🏠 Menú principal", use_container_width=True):
            st.session_state.mode = "MENU"
            st.rerun()

# ===================== PENALTIS =====================
def reset_penaltis():
    st.session_state.pen_intentos = 5
    st.session_state.pen_goles = 0
    st.session_state.pen_cpu = 0
    st.session_state.pen_sudden = False

def pen_text():
    return f"Intentos: {st.session_state.pen_intentos} · Marcador — Tú {st.session_state.pen_goles} : {st.session_state.pen_cpu} CPU"

def pen_tirar(eleccion):
    if st.session_state.pen_intentos <= 0 and not st.session_state.pen_sudden:
        return
    portero = random.choice(["Izquierda", "Centro", "Derecha"])
    cpu_tiro = random.choice(["Izquierda", "Centro", "Derecha"])

    msg_user = "¡GOL! " if eleccion != portero else "¡Atajado! "
    msg_user += f"(Portero: {portero} / Tú: {eleccion})"
    if eleccion != portero:
        st.session_state.pen_goles += 1

    portero_tu = random.choice(["Izquierda", "Centro", "Derecha"])
    if cpu_tiro != portero_tu:
        st.session_state.pen_cpu += 1
        msg_cpu = f"CPU anotó (CPU: {cpu_tiro} / Tu portero: {portero_tu})"
    else:
        msg_cpu = f"¡Tu portero atajó! (CPU: {cpu_tiro} / Portero: {portero_tu})"

    if not st.session_state.pen_sudden:
        st.session_state.pen_intentos -= 1

    st.session_state.pen_last = f"{msg_user}\n{msg_cpu}"

    # Chequear fin
    if st.session_state.pen_intentos == 0 and not st.session_state.pen_sudden:
        if st.session_state.pen_goles == st.session_state.pen_cpu:
            st.session_state.pen_sudden = True
            st.info("🤯 ¡Muerte súbita! Sigue tirando hasta desempatar.")
        else:
            st.session_state.mode = "PENALTIS_RESULT"

    if st.session_state.pen_sudden:
        # muerte súbita — termina si hay diferencia
        if st.session_state.pen_goles != st.session_state.pen_cpu:
            st.session_state.mode = "PENALTIS_RESULT"

def view_penaltis():
    st.markdown(f"<div class='title'>⚽ Penaltis</div>", unsafe_allow_html=True)
    st.markdown(f"<div class='subtitle'>{pen_text()}</div>", unsafe_allow_html=True)
    st.write("")
    c = st.columns(3)
    with c[0]:
        st.button("↖ Izquierda", use_container_width=True, key="L", on_click=lambda: pen_tirar("Izquierda"))
    with c[1]:
        st.button("⬆ Centro", use_container_width=True, key="C", on_click=lambda: pen_tirar("Centro"))
    with c[2]:
        st.button("↗ Derecha", use_container_width=True, key="R", on_click=lambda: pen_tirar("Derecha"))
    st.write("")
    st.info(st.session_state.get("pen_last", "Elige tu tiro"))
    st.write("")
    if st.button("🏠 Menú", use_container_width=True):
        st.session_state.mode = "MENU"
        st.rerun()

def view_penaltis_result():
    res = "🏆 ¡Ganaste!" if st.session_state.pen_goles > st.session_state.pen_cpu else ("🤝 Empate (raro aquí)" if st.session_state.pen_goles==st.session_state.pen_cpu else "😢 Perdiste")
    st.markdown(f"<div class='title'>{res}</div>", unsafe_allow_html=True)
    st.markdown(f"<div class='card'><div class='question'>Marcador final: Tú {st.session_state.pen_goles} — {st.session_state.pen_cpu} CPU</div></div>", unsafe_allow_html=True)
    c1, c2 = st.columns(2)
    with c1:
        if st.button("🔁 Jugar de nuevo", use_container_width=True):
            reset_penaltis()
            st.session_state.mode = "PENALTIS"
            st.rerun()
    with c2:
        if st.button("🏠 Menú", use_container_width=True):
            st.session_state.mode = "MENU"
            st.rerun()

# ===================== ROUTER =====================
if st.session_state.mode == "MENU":
    view_menu()
elif st.session_state.mode == "TRIVIA":
    # Ensure keys exist if user recarga
    if not st.session_state.preguntas:
        st.session_state.mode = "MENU"
        st.rerun()
    view_trivia_real()
elif st.session_state.mode == "RESULT":
    view_result()
elif st.session_state.mode == "PENALTIS":
    view_penaltis()
elif st.session_state.mode == "PENALTIS_RESULT":
    view_penaltis_result()
else:
    st.session_state.mode = "MENU"
    st.rerun()

