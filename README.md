# SQL-MariaDB_10.2-
/*Triggers and Procedures
PROCEDIMIENTO: Ejecuta bucle para conjunto de resultados.
  1- Declaración de CURSOR para almacenar conjunto de resultado 
  2- Esctuctura read_loop: LOOP para interactuar con CURSOR y ejecutar setencia para cada fila del cursor
  3- Realizar UPDATE con por cada fila contenida en CURSOR
  4- Implementar TRIGGER para enlazar evento UPDATE de la tabla al PROCEDURE */

/************************************************************************************************************************/
          PROCEDURE
/************************************************************************************************************************/
CREATE PROCEDURE SP_ReponerStockDeposito(IN idventa INT(11), IN iddeposito INT(3))
BEGIN
    DECLARE finalizar INT DEFAULT FALSE; 
    DECLARE codProd CHAR(60);
    DECLARE cantVen DECIMAL(12,3);
    DECLARE existe_stock  INT DEFAULT 0;

    DECLARE cursordetalle CURSOR FOR SELECT prd_id, vendt_can FROM ventas_detalle WHERE vent_id= idventa;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET finalizar = TRUE;

    OPEN cursordetalle;

    read_loop: LOOP 
        FETCH cursordetalle INTO codProd,cantVen;
    
        IF finalizar THEN 
            LEAVE read_loop;
        END IF;
    
        set existe_stock = 0;
        SELECT 1 INTO existe_stock FROM stock WHERE prd_id= codProd AND dep_id= iddeposito;
        IF existe_stock = 1 THEN
            UPDATE stock SET stk_canactual = stk_canactual + cantVen WHERE  prd_id = codProd AND dep_id = iddeposito;
        END IF;
    END LOOP;
    CLOSE cursordetalle;
END;



/************************************************************************************************************************/
          TRIGGER
/************************************************************************************************************************/

CREATE DEFINER=`root`@`localhost` TRIGGER TRG_ReponerStockVentaAnulado 
    AFTER UPDATE 
    ON ventas FOR EACH ROW 
    BEGIN 
        
        DECLARE idDeposito INT; 

        DECLARE idVenta INT; 
        
        SELECT dep_id INTO idDeposito FROM cajas where caj_id= OLD.caj_id;

        set idVenta = OLD.vent_id;
        IF NEW.vent_anulado=1 THEN
            CALL SP_ReponerStockDeposito(idVenta, idDeposito);
        END IF;

    END; 
