from PIL import Image

# ----------- Текстті бинарлы форматқа түрлендіру -----------
def text_to_binary(text):
    return ''.join(format(ord(char), '08b') for char in text)

# ----------- Бинарлы кодты мәтінге түрлендіру -----------
def binary_to_text(binary_data):
    chars = [binary_data[i:i+8] for i in range(0, len(binary_data), 8)]
    message = ''
    for char in chars:
        if char == '11111110':  # Соңғы маркер
            break
        message += chr(int(char, 2))
    return message

# ----------- Ақпаратты суретке жасыру -----------
def encode_image(image_path, message, output_path):
    img = Image.open(image_path)
    binary_msg = text_to_binary(message) + '11111110'  # Соңына маркер қосу
    img = img.convert('RGB')
    pixels = list(img.getdata())

    encoded_pixels = []
    binary_index = 0

    for pixel in pixels:
        r, g, b = pixel
        if binary_index < len(binary_msg):
            r = (r & ~1) | int(binary_msg[binary_index])
            binary_index += 1
        if binary_index < len(binary_msg):
            g = (g & ~1) | int(binary_msg[binary_index])
            binary_index += 1
        if binary_index < len(binary_msg):
            b = (b & ~1) | int(binary_msg[binary_index])
            binary_index += 1
        encoded_pixels.append((r, g, b))

    img.putdata(encoded_pixels)
    img.save(output_path)
    print(f"[+] Хабарлама сәтті '{output_path}' суретіне жасырылды.")

# ----------- Ақпаратты суреттен шығару -----------
def decode_image(image_path):
    img = Image.open(image_path)
    img = img.convert('RGB')
    pixels = list(img.getdata())

    binary_data = ''
    for pixel in pixels:
        for color in pixel:
            binary_data += str(color & 1)

    message = binary_to_text(binary_data)
    print(f"[+] Жасырылған хабар: {message}")
    return message

# ----------- Негізгі мәзір -----------
def main():
    print("BMP файлында ақпаратты жасыру (LSB әдісі)")
    print("1. Хабарды суретке жасыру")
    print("2. Суреттен хабарды шығару")
    choice = input("Таңдауыңызды енгізіңіз (1/2): ")

    if choice == '1':
        img_path = input("Бастапқы BMP файл атауы: ")
        secret_msg = input("Жасыратын хабар: ")
        output_img = input("Нәтижелік BMP файл атауы: ")
        encode_image(img_path, secret_msg, output_img)
    elif choice == '2':
        img_path = input("Хабар жасырылған BMP файл атауы: ")
        decode_image(img_path)
    else:
        print("Қате таңдау!")

if __name__ == "__main__":
    main()
