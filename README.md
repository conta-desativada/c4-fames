# c4-fames
import pygame
import sys
import random

pygame.init()

info = pygame.display.Info()
LARGURA = info.current_w
ALTURA = info.current_h

tela = pygame.display.set_mode((LARGURA, ALTURA), pygame.FULLSCREEN)
pygame.display.set_caption("Desarme a Bomba")

# Cores
PRETO = (12, 12, 12)
BRANCO = (255, 255, 255)
CINZA = (60, 60, 60)
VERMELHO = (220, 40, 40)
VERDE = (40, 180, 60)
AZUL = (40, 110, 230)
AMARELO = (255, 210, 40)
VERDE_BOMBA = (46, 125, 50)
VERDE_ESCURO = (30, 90, 35)
CINZA_PAINEL = (45, 45, 50)
CINZA_CLARO = (70, 70, 75)

fonte_grande = pygame.font.SysFont("Arial", 42, bold=True)
fonte_media = pygame.font.SysFont("Arial", 30, bold=True)
fonte_pequena = pygame.font.SysFont("Arial", 24)
fonte_timer = pygame.font.SysFont("Courier New", 36, bold=True)

FIOS = [
    {"nome": "AZUL", "cor": AZUL},
    {"nome": "VERDE", "cor": VERDE},
    {"nome": "VERMELHO", "cor": VERMELHO},
    {"nome": "PRETO", "cor": (40, 40, 40)}
]

class Botao:
    def __init__(self, x, y, w, h, texto, cor):
        self.rect = pygame.Rect(x, y, w, h)
        self.texto = texto
        self.cor = cor
        self.cortado = False
        self.ativo = True

    def desenhar(self, superficie):
        cor = CINZA if (self.cortado or not self.ativo) else self.cor
        pygame.draw.rect(superficie, cor, self.rect, border_radius=14)
        
        if self.texto == "PRETO" and not self.cortado:
            pygame.draw.rect(superficie, (160, 160, 160), self.rect, 3, border_radius=14)
        else:
            pygame.draw.rect(superficie, BRANCO, self.rect, 3, border_radius=14)

        txt = fonte_media.render(self.texto, True, BRANCO)
        superficie.blit(txt, txt.get_rect(center=self.rect.center))

    def foi_clicado(self, pos):
        return self.rect.collidepoint(pos) and self.ativo and not self.cortado

def desenhar_bomba(superficie, tempo, estado):
    cx = LARGURA // 2
    cy = ALTURA // 3.3

    # Corpo principal verde
    corpo = pygame.Rect(cx - 110, cy - 50, 220, 115)
    pygame.draw.rect(superficie, VERDE_BOMBA, corpo, border_radius=14)
    pygame.draw.rect(superficie, VERDE_ESCURO, corpo, 4, border_radius=14)

    # Painel do display (mais no meio da bomba)
    painel = pygame.Rect(cx - 85, cy - 25, 170, 45)
    pygame.draw.rect(superficie, CINZA_PAINEL, painel, border_radius=8)
    pygame.draw.rect(superficie, CINZA_CLARO, painel, 2, border_radius=8)

    # Fundo preto do número
    display = pygame.Rect(cx - 70, cy - 18, 140, 32)
    pygame.draw.rect(superficie, (15, 15, 15), display, border_radius=5)

    # Texto do timer
    if estado == "explodiu":
        texto_timer = "BOOM"
        cor_timer = VERMELHO
    elif estado == "desarmada":
        texto_timer = "SAFE"
        cor_timer = VERDE
    else:
        m = tempo // 60
        s = tempo % 60
        texto_timer = f"{m:02d}:{s:02d}"
        cor_timer = VERMELHO

    txt = fonte_timer.render(texto_timer, True, cor_timer)
    superficie.blit(txt, txt.get_rect(center=display.center))

    # LED vermelho
    pygame.draw.circle(superficie, VERMELHO, (cx + 72, cy - 5), 5)

    # Detalhe superior
    pygame.draw.rect(superficie, (30, 30, 30), (cx - 12, cy - 58, 24, 6), border_radius=2)

def main():
    relogio = pygame.time.Clock()

    while True:
        fio_correto = random.choice(FIOS)
        tempo = 60
        ultimo = pygame.time.get_ticks()
        estado = "jogando"
        mensagem = "Corte o fio correto!"

        botoes_fios = []
        altura_botao = 75
        espaco = 12
        inicio_y = ALTURA - (4 * (altura_botao + espaco)) - 30

        for i, fio in enumerate(FIOS):
            y = inicio_y + i * (altura_botao + espaco)
            botoes_fios.append(Botao(20, y, LARGURA - 40, altura_botao, fio["nome"], fio["cor"]))

        botao_reiniciar = Botao(
            40, ALTURA // 2 + 90,
            LARGURA - 80, 85,
            "REINICIAR JOGO", (0, 150, 255)
        )
        botao_reiniciar.ativo = False

        rodando_partida = True
        while rodando_partida:
            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if evento.type == pygame.KEYDOWN and evento.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()

                if evento.type == pygame.MOUSEBUTTONDOWN:
                    pos = evento.pos

                    if estado == "jogando":
                        for botao in botoes_fios:
                            if botao.foi_clicado(pos):
                                botao.cortado = True
                                if botao.texto == fio_correto["nome"]:
                                    estado = "desarmada"
                                    mensagem = "BOMBA DESARMADA!"
                                    botao_reiniciar.ativo = True
                                else:
                                    estado = "explodiu"
                                    mensagem = f"Fio {botao.texto} era o errado!"
                                    botao_reiniciar.ativo = True

                    if botao_reiniciar.foi_clicado(pos):
                        rodando_partida = False

            if estado == "jogando":
                agora = pygame.time.get_ticks()
                if agora - ultimo >= 1000:
                    tempo -= 1
                    ultimo = agora
                    if tempo <= 0:
                        estado = "explodiu"
                        mensagem = "Tempo esgotado!"
                        botao_reiniciar.ativo = True

            tela.fill(PRETO)

            titulo = fonte_media.render("DESARME A BOMBA", True, BRANCO)
            tela.blit(titulo, (20, 20))

            cor_msg = AMARELO if estado == "jogando" else (VERDE if estado == "desarmada" else VERMELHO)
            msg = fonte_pequena.render(mensagem, True, cor_msg)
            tela.blit(msg, (20, 65))

            desenhar_bomba(tela, tempo, estado)

            if estado == "jogando":
                for botao in botoes_fios:
                    botao.desenhar(tela)
            else:
                botao_reiniciar.desenhar(tela)

            pygame.display.flip()
            relogio.tick(60)

if __name__ == "__main__":
    main()