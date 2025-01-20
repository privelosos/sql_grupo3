drop TABLE demo

-- Tabela Clientes
CREATE TABLE Clientes (
    ID_Cliente SERIAL PRIMARY KEY,
    Nome VARCHAR(255),
    Endereco VARCHAR(255),
    Telefone VARCHAR(20)
);

INSERT INTO Clientes (Nome, Endereco, Telefone) VALUES
('João Silva', 'Rua Paraiba, 23', '67999998888'),
('Maria Oliveira', 'Rua Dona Bia Taveira, 456', '67988887777'),
('Carlos Santos', 'Rua Espirito Santo, 789', '67977776666'),
('Ana Souza', 'Rua Itiquira, 176', '67966665555'),
('Pedro Lima', 'Rua Nortelândia, 421', '67955554444');

SELECT * FROM Clientes;

-- Tabela Pedidos
CREATE TABLE Pedidos (
    ID_Pedido SERIAL PRIMARY KEY,
    ID_Cliente INT,
    Data DATE,
    Total DECIMAL(10,2),
    FOREIGN KEY (ID_Cliente) REFERENCES Clientes(ID_Cliente)
);

-- Inserindo dados na tabela Pedidos (apenas ID_Pedido e ID_Cliente)
INSERT INTO Pedidos (ID_Cliente, Data) VALUES
(1, CURRENT_DATE),  -- Pedido de João Silva
(2, CURRENT_DATE),  -- Pedido de Maria Oliveira
(3, CURRENT_DATE),  -- Pedido de Carlos Santos
(4, CURRENT_DATE);  -- Pedido de Ana Souza


-- Tabela Pizzas
CREATE TABLE Pizzas (
    ID_Pizza SERIAL PRIMARY KEY,
    Sabor VARCHAR(255),
    Preco DECIMAL(10,2)
);

INSERT INTO Pizzas (Sabor, Preco) VALUES
('Margherita Grande', 106.00),
('Margherita Pequena', 74.20),
('Margherita Speciale Grande', 114.00),
('Margherita Speciale Pequena', 79.80),
('Nostra Burrata Grande', 112.00),
('Nostra Burrata Pequena', 78.40),
('Napoletana Grande', 106.00),
('Napoletana Pequena', 74.20),
('Alici Grande', 117.00),
('Alici Pequena', 81.90),
('Margherita Grande Integral', 116.00),
('Margherita Speciale Pequena Sem Glúten', 89.80),
('Meio a Meio Margherita/Napoletana Grande', 106.00);

SELECT * FROM Pizzas;

-- Tabela Bebidas
CREATE TABLE Bebidas (
    ID_bebidas SERIAL PRIMARY KEY,
    Tipo VARCHAR(255),
    Preco DECIMAL(10,2)
);

INSERT INTO Bebidas (Tipo, Preco) VALUES
('Cerveja (600 ml)', 8.50),
('Água (com gás)', 3.00),
('Água (sem gás)', 2.50),
('Refrigerante (lata)', 4.00),
('Suco natural', 6.00);

SELECT * FROM Bebidas;

-- Tabela Pedidos_Pizzas (tabela intermediária para o relacionamento N:N)
CREATE TABLE Pedidos_Pizzas (
    ID_Pedido INT,
    ID_Pizza INT,
    FOREIGN KEY (ID_Pedido) REFERENCES Pedidos(ID_Pedido),
    FOREIGN KEY (ID_Pizza) REFERENCES Pizzas(ID_Pizza)
);

-- Tabela Pedidos_Bebidas (tabela intermediária para o relacionamento N:N)
CREATE TABLE Pedidos_Bebidas (
    ID_Pedido INT,
    ID_bebidas INT,
    FOREIGN KEY (ID_Pedido) REFERENCES Pedidos(ID_Pedido),
    FOREIGN KEY (ID_bebidas) REFERENCES Bebidas(ID_bebidas)
);

-- Inserindo Pizzas nos Pedidos
INSERT INTO Pedidos_Pizzas (ID_Pedido, ID_Pizza) VALUES
(1, 1), -- Pedido 1 (João Silva) inclui Margherita Grande
(1, 2), -- Pedido 1 (João Silva) inclui Margherita Pequena
(2, 3), -- Pedido 2 (Maria Oliveira) inclui Margherita Speciale Grande
(3, 4), -- Pedido 3 (Carlos Santos) inclui Margherita Speciale Pequena
(4, 5); -- Pedido 4 (Ana Souza) inclui Nostra Burrata Grande

-- Inserindo Bebidas nos Pedidos
INSERT INTO Pedidos_Bebidas (ID_Pedido, ID_bebidas) VALUES
(1, 1), -- Pedido 1 (João Silva) inclui uma Cerveja (600 ml)
(1, 4), -- Pedido 1 (João Silva) inclui um Refrigerante (lata)
(2, 2), -- Pedido 2 (Maria Oliveira) inclui uma Água (com gás)
(3, 5), -- Pedido 3 (Carlos Santos) inclui um Suco natural
(4, 3); -- Pedido 4 (Ana Souza) inclui uma Água (sem gás)

-- Atualizar os totais na tabela Pedidos
UPDATE Pedidos
SET Total = (
    SELECT SUM(P.Preco) 
    FROM Pedidos_Pizzas PP
    JOIN Pizzas P ON PP.ID_Pizza = P.ID_Pizza
    WHERE PP.ID_Pedido = Pedidos.ID_Pedido
) + (
    SELECT SUM(B.Preco) 
    FROM Pedidos_Bebidas PB
    JOIN Bebidas B ON PB.ID_bebidas = B.ID_bebidas
    WHERE PB.ID_Pedido = Pedidos.ID_Pedido
);

SELECT * FROM Pedidos;

-- 1. Consultar todos os clientes que fizeram pedidos acima de um valor específico
SELECT c.Nome AS Cliente, p.Total
FROM Pedidos p
JOIN Clientes c ON p.ID_Cliente = c.ID_Cliente
WHERE p.Total > 100.00

-- 2. Consultar todas as pizzas vendidas e suas quantidades
SELECT pi.Sabor, COUNT(pp.ID_Pizza) AS Quantidade_Vendida
FROM Pedidos_Pizzas pp
JOIN Pizzas pi ON pp.ID_Pizza = pi.ID_Pizza
GROUP BY pi.Sabor
ORDER BY Quantidade_Vendida DESC;
