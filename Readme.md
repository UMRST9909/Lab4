import random

def generate_s_box(size):
    """ Генерує випадкову таблицю заміни для S-блоку заданого розміру. """
    values = list(range(size))
    random.shuffle(values)
    s_box = {i: values[i] for i in range(size)}
    reverse_s_box = {v: k for k, v in s_box.items()}
    return s_box, reverse_s_box

def s_box_transform(data, s_box):
    """ Пряме перетворення через S-блок. """
    high_nibble = (data >> 4) & 0b1111
    low_nibble = data & 0b1111
    return (s_box[high_nibble] << 4) | s_box[low_nibble]

def s_box_reverse_transform(data, reverse_s_box):
    """ Зворотне перетворення через S-блок. """
    high_nibble = (data >> 4) & 0b1111
    low_nibble = data & 0b1111
    return (reverse_s_box[high_nibble] << 4) | reverse_s_box[low_nibble]

# Випадкова перестановка для P-блоку
def generate_p_box_permutation(size):
    """ Генерує випадкову перестановку для P-блоку заданого розміру. """
    permutation = list(range(size))
    random.shuffle(permutation)
    return permutation

def p_box_permute(data, permutation):
    """ Пряме перетворення через P-блок. """
    permuted_data = 0
    for i, bit_position in enumerate(permutation):
        bit = (data >> bit_position) & 1
        permuted_data |= (bit << (7 - i))
    return permuted_data

def p_box_reverse_permute(data, permutation):
    """ Зворотне перетворення через P-блок. """
    reverse_permutation = [permutation.index(i) for i in range(8)]
    reverse_data = 0
    for i, bit_position in enumerate(reverse_permutation):
        bit = (data >> (7 - i)) & 1
        reverse_data |= (bit << bit_position)
    return reverse_data

# Генеруємо S-блок і P-блок
S_BOX_SIZE = 16  # Для 4-бітних тетрад
s_box, reverse_s_box = generate_s_box(S_BOX_SIZE)
p_box_permutation = generate_p_box_permutation(8)

def encrypt(data):
    """ Шифрування: послідовне застосування S-блоку та P-блоку. """
    s_transformed = s_box_transform(data, s_box)
    p_permuted = p_box_permute(s_transformed, p_box_permutation)
    return p_permuted

def decrypt(data):
    """ Розшифрування: послідовне застосування зворотного P-блоку та S-блоку. """
    p_reversed = p_box_reverse_permute(data, p_box_permutation)
    s_reversed = s_box_reverse_transform(p_reversed, reverse_s_box)
    return s_reversed

# Тестування з різними вхідними даними
test_data = [0b11001100, 0b10101010, 0b11110000, 0b00011111]
for original_data in test_data:
    encrypted_data = encrypt(original_data)
    decrypted_data = decrypt(encrypted_data)
    print(f"Оригінальні дані: {bin(original_data)}")
    print(f"Зашифровані дані: {bin(encrypted_data)}")
    print(f"Розшифровані дані: {bin(decrypted_data)}")
    print("-" * 30)