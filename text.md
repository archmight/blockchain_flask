#%% md

# Блокчейн

#%% md

Ниже рассматривается пример блокчейна на языке python

блокчейн состоит из 2-х основных структур:

1. цепочка блоков
2. список транзакций текущего блока



#%%

class Blockchain(object):

    def __init__(self):
        self.chain = []
        self.current_transactions = []

    def new_block(self):
        # Создает новый блок и вносит его в цепь
        pass

    def new_transaction(self):
        # Вносит новую транзакцию в список транзакций
        pass

    @staticmethod
    def hash(block):
        # Хеширует блок
        pass

    @property
    def last_block(self):
        # Возвращает последний блок в цепочке
        pass



#%% md

В данном примере кода на python хранится вся цепочка целиком

рассмотрим каждый метод в отдельности:

#%% md

### new transaction

#%%

def new_transaction(self, sender, recipient, amount):
    """
    Направляет новую транзакцию в следующий блок

    :param sender: <str> Адрес отправителя
    :param recipient: <str> Адрес получателя
    :param amount: <int> Сумма
    :return: <int> Индекс блока, который будет хранить эту транзакцию
    """

    self.current_transactions.append({
        'sender': sender,
        'recipient': recipient,
        'amount': amount,
    })

    return self.last_block['index'] + 1


#%% md

Метод *new transaction* вносит транзакцию в список транзакций и возвращает индекс блока в который будет осуществлена запись транзакции;

#%% md

### new block

#%%

from time import time

def new_block(self, proof, previous_hash=None):
    """
    Создание нового блока в блокчейне

    :param proof: <int> Доказательства проведенной работы
    :param previous_hash: (Опционально) хеш предыдущего блока
    :return: <dict> Новый блок
    """

    block = {
        'index': len(self.chain) + 1,
        'timestamp': time(),
        'transactions': self.current_transactions,
        'proof': proof,
        'previous_hash': previous_hash or self.hash(self.chain[-1]),
    }

    # Перезагрузка текущего списка транзакций
    self.current_transactions = []

    self.chain.append(block)
    return block


#%% md

Метод *new block* формирует новый блок, очищает список транзакции для нового блока и присоединяет созданный блок к цепочке


#%% md

### hash

#%%

import json
import hashlib
@staticmethod
def hash(block):
    """
    Создает хэш SHA-256 блока

    :param block: <dict> Блок
    :return: <str>
    """

    # Мы должны убедиться в том, что словарь упорядочен, иначе у нас будут непоследовательные хеши
    block_string = json.dumps(block, sort_keys=True).encode()
    return hashlib.sha256(block_string).hexdigest()

#%% md

В методе *hash* осуцествляется хэширование блока

#%%

### Структура блока
block = {
    'index': 1,
    'timestamp': 1506057125.900785,
    'transactions': [
        {
            'sender': "8527147fe1f5426f9dd545de4b27ee00",
            'recipient': "a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount': 5,
        }
    ],
    'proof': 324984774000,
    'previous_hash': "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}

#%% md

Каждый блок содержит индекс, временной штамп (время unix), список транзакций, доказательство и хеш предыдущего блока.

### Иммутабельность блокчейна обеспечивается  наличием хэша прэдыдущего блока



#%% md


# PoW(1993)
принцип защиты сетевых систем, основанный на необходимости выполнения на стороне клиента некоторой достаточно длительной работы (нахождение решения задачи), результат которой легко и быстро проверяется на стороне сервера

# hashcash (1997)
алгоритм построенный на основе *PoW*, используемый во множестве криптовалют.

#%%

import matplotlib.pyplot as plt
import matplotlib.image as mpimg
img = mpimg.imread('img.png')
imgplot = plt.imshow(img)
plt.show()

#%% md

Пример заголовка hashcash

#%% md



#%% md

### на клиентской стороне
Отправитель подготавливает заголовок и добавляет к нему случайное число до тех пор, пока не будет получена указанная последовательность нулей вначале, инкрементируя значения счетчика

в данном случае вероятность составит 1/2^20 ~ 1/10^6, что несущественно замедлит работу пользователя, однако осложнит деятельность спамеров

### на стороне сервера
Если первые 20 бит ненулевые, хеш является недействительным

Если разница с текущей датой более двух дней, хеш является недействительным

Если e-mail в строке хеша не свопадает ни с каким e-mail адресом,
зарегистрированным получателем или с любым адресом из списка тех, на которые получатель подписан, хеш является недействительным

### Если такая хэш строка присутствует в базе  -  хеш является недействительным

 в отличие от почты, где используются 20 бит (порядка 1 млн попыток для успешного поиска), биткойн использует 67,5 бит (необходимо порядка 200 млн триллионов попыток) и меняющийся критерий сложности, чтобы сгенерировать один из блоков, которые создаются каждые 10 минут. В биткойне скорректировали алгоритм, добавив поддержку работы с долями бит (первоначальная спецификация HashCash ограничивалась корректировкой целых степеней числа 2), тем самым удалось достичь более высокой точности.

#%%

import hashlib
import json

from time import time
from uuid import uuid4


class Blockchain(object):
    ...
        
    def proof_of_work(self, last_proof):
        """
        Простая проверка алгоритма:
         - Поиска числа p`, так как hash(pp`) содержит 4 заглавных нуля, где p - предыдущий
         - p является предыдущим доказательством, а p` - новым

        :param last_proof: <int>
        :return: <int>
        """

        proof = 0
        while self.valid_proof(last_proof, proof) is False:
            proof += 1

        return proof

    @staticmethod
    def valid_proof(last_proof, proof):
        """
        Подтверждение доказательства: Содержит ли hash(last_proof, proof) 4 заглавных нуля?

        :param last_proof: <int> Предыдущее доказательство
        :param proof: <int> Текущее доказательство
        :return: <bool> True, если правильно, False, если нет.
        """

        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"

#%% md

Чтобы скорректировать сложность алгоритма, мы можем изменить количество заглавных нулей. В нашем случае, 4 — достаточно. Внесение одного ведущего нуля создает колоссальную разницу во времени, необходимом для поиска решения (майнинга).

#%% md

# REST API

#%% md

В качестве протокола для блокчейна будет использовать http, а для сериализации json

#%% md

единственным узлом сети будет сам сервер

#%% md

создадим 3 метода:
* для вызова создания новой транзакции в блоке
* для майнинга нового блока сервером
* для получения блокчейна

#%% md

### /transactions/new

#%%


@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    values = request.get_json()

    # Убедитесь в том, что необходимые поля находятся среди POST-данных
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Missing values', 400

    # Создание новой транзакции
    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])

    response = {'message': f'Transaction will be added to Block {index}'}
    return jsonify(response), 201

#%% md

### /mine

#%%

@app.route('/mine', methods=['GET'])
def mine():
    # Мы запускаем алгоритм подтверждения работы, чтобы получить следующее подтверждение…
    last_block = blockchain.last_block
    last_proof = last_block['proof']
    proof = blockchain.proof_of_work(last_proof)

    # Мы должны получить вознаграждение за найденное подтверждение
    # Отправитель “0” означает, что узел заработал крипто-монету
    blockchain.new_transaction(
        sender="0",
        recipient=node_identifier,
        amount=1,
    )

    # Создаем новый блок, путем внесения его в цепь
    previous_hash = blockchain.hash(last_block)
    block = blockchain.new_block(proof, previous_hash)

    response = {
        'message': "New Block Forged",
        'index': block['index'],
        'transactions': block['transactions'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash'],
    }
    return jsonify(response), 200


#%% md

### /chain

#%%

@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200
