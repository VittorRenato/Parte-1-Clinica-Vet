# Parte-1-Clinica-Vet


create database clinica_veterinaria;
use clinica_veterinaria;


CREATE TABLE pacientes (
    id_paciente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    especie VARCHAR(50) NOT NULL,
	idade INT
);

INSERT INTO Pacientes (nome, especie, idade) VALUES 
('Renatto', 'Gato', 23),
SELECT * FROM Pacientes;

CREATE TABLE veterinarios (
    id_veterinario  INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    especialidade VARCHAR(50)
);
INSERT INTO Veterinarios (nome, especialidade) VALUES 
('Vittor', 'Caninos');

select * from veterinarios;

CREATE TABLE consultas (
    id_consulta INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente  INT NOT NULL,
    id_veterinario INT NOT NULL,
    data_consulta DATE NOT NULL,
	custo DECIMAL(10,2),
    FOREIGN KEY (id_paciente) REFERENCES Pacientes(id_paciente),
    FOREIGN KEY (id_veterinario ) REFERENCES Veterinarios(id_veterinario)
);	

insert into consultas values(1, 1, 1, '2024-09-24', 130.00);
select * from consultas;






DELIMITER //

CREATE PROCEDURE agendar_consulta(
    IN sp_id_paciente INT,
    IN sp_id_veterinario INT,
    IN sp_data_consulta DATE,
    IN sp_custo DECIMAL(10,2)
)
BEGIN
    INSERT INTO Consultas (id_paciente, id_veterinario, data_consulta, custo)
    VALUES (sp_id_paciente, sp_id_veterinario, sp_data_consulta, sp_custo);
END //

DELIMITER ;

call agendar_consulta(1,1,'2024-09-24',300.00);


DELIMITER //

CREATE PROCEDURE atualizar_paciente(
    IN sp_id_paciente INT,
    IN sp_novo_nome VARCHAR(100),
    IN sp_nova_especie VARCHAR(50),
    IN sp_nova_idade INT
)
BEGIN
    UPDATE pacientes
    SET nome = sp_novo_nome,
        especie = sp_nova_especie,
        idade = sp_nova_idade
    WHERE id_paciente = sp_id_paciente;
END //

DELIMITER ;


CALL atualizar_paciente(1, 'Vittor Renato', 'Gato', 12);


DELIMITER //




CREATE PROCEDURE remover_consulta(
    IN sp_id_consulta INT
)
BEGIN
    DELETE FROM consultas
    WHERE id_consulta = sp_id_consulta;
END //

DELIMITER ;


CALL remover_consulta(1);


DELIMITER //

CREATE FUNCTION total_gasto_paciente(
    sp_id_paciente INT
) 
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);

    SELECT COALESCE(SUM(custo), 0) INTO total
    FROM consultas
    WHERE id_paciente = p_id_paciente;

    RETURN total;
END //

DELIMITER ;


SELECT total_gasto_paciente(1) AS total_gasto;




DELIMITER //

CREATE TRIGGER verificar_idade_paciente
BEFORE INSERT ON pacientes
FOR EACH ROW
BEGIN
    IF NEW.idade < 0 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Idade inválida: digite uma idade positiva.';
    END IF;
END //

DELIMITER ;


INSERT INTO pacientes (nome, especie, idade) VALUES ('Murilo', 'Jacare', -330);




CREATE TABLE Log_Consultas (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    id_consulta INT,
    custo_antigo DECIMAL(10, 2),
    custo_novo DECIMAL(10, 2)
);
 
 
 DELIMITER //

CREATE TRIGGER atualizar_custo_consulta
AFTER UPDATE ON consultas
FOR EACH ROW
BEGIN
    IF OLD.custo <> NEW.custo THEN
        INSERT INTO Log_Consultas (id_consulta, custo_antigo, custo_novo)
        VALUES (NEW.id_consulta, OLD.custo, NEW.custo);
    END IF;
END //

DELIMITER ;


UPDATE consultas SET custo = 69.69 WHERE id_consulta = 1;


SELECT * FROM Log_Consultas;
