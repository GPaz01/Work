import qrcode
from PIL import Image, ImageDraw, ImageFont


def criar_etiqueta_patrimonio(
    url, id, ficheiro_saida, logo_path=None
):
    # 1. Configurar o QR Code com tolerância alta a erros (ERROR_CORRECT_H = 30%)
    # Isso permite cobrir o centro com a logo sem perder a capacidade de leitura.
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_H,
        box_size=10,
        border=2,
    )
    qr.add_data(url)
    qr.make(fit=True)

    # Gerar a imagem base do QR Code com canal de transparência (RGBA)
    qr_img = qr.make_image(fill_color="black", back_color="white").convert(
        "RGBA"
    )

    # 2. Inserir a Logo no Centro
    if logo_path:
        try:
            logo = Image.open(logo_path).convert("RGBA")
            qr_w, qr_h = qr_img.size

            # Ajusta o tamanho da logo para ~22% da largura total do QR Code
            logo_size = int(qr_w * 0.22)
            logo = logo.resize((logo_size, logo_size), Image.LANCZOS)

            # Cria uma moldura/fundo branco atrás da logo para isolá-la do código
            fundo_logo = Image.new(
                "RGBA", (logo_size + 6, logo_size + 6), "white"
            )
            pos_fundo = (
                (qr_w - logo_size - 6) // 2,
                (qr_h - logo_size - 6) // 2,
            )
            qr_img.paste(fundo_logo, pos_fundo)

            # Cola a logo centralizada
            pos_logo = ((qr_w - logo_size) // 2, (qr_h - logo_size) // 2)
            qr_img.paste(logo, pos_logo, mask=logo)
        except Exception as e:
            print(
                f"⚠️ Não foi possível carregar a logo ({e}). A gerar sem logo..."
            )

    # Converter para formato RGB
    qr_img = qr_img.convert("RGB")

    # 3. Adicionar o espaço inferior para o código do Património
    largura_qr, altura_qr = qr_img.size
    altura_legenda = 45

    etiqueta_final = Image.new(
        "RGB", (largura_qr, altura_qr + altura_legenda), color="white"
    )
    etiqueta_final.paste(qr_img, (0, 0))

    # 4. Escrever o texto do Património centralizado
    draw = ImageDraw.Draw(etiqueta_final)
    font = ImageFont.load_default()

    bbox = draw.textbbox((0, 0), id, font=font)
    texto_largura = bbox[2] - bbox[0]
    pos_x = (largura_qr - texto_largura) // 2
    pos_y = altura_qr + 12

    draw.text((pos_x, pos_y), id, fill="black", font=font)

    # 5. Guardar a imagem final
    etiqueta_final.save(ficheiro_saida)
    print(f"✅ Etiqueta gerada com sucesso: {ficheiro_saida}")


# ==========================================
# 🛠️ EXECUÇÃO COM A SUA LOGO
# ==========================================
# loop para fazer varios QRcodes dentro de uma pasta em especifico
for i in range(1, 200):
    criar_etiqueta_patrimonio(
        url="https://glpi.services-apps.com.br/front/computer.form.php?id="+str(i),
        id="ID: #"+str(i),
        ficheiro_saida="qrcode/qrcode_"+str(i)+".png",
        logo_path="logo.png",  # Nome do ficheiro da logo guardado na mesma pasta

    )
