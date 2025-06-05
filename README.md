# SQL-RESTOR-AUTOMATIC
A dynamic SQL Server script to automatically restore multiple databases from .bak files using RESTORE FILELISTONLY. Avoids manual logical file name mapping. Ideal for dev, QA, or CI/CD environments.
DECLARE @BackupPath NVARCHAR(260) = 'D:\AutoBackUp\';
DECLARE @DataPath NVARCHAR(260) = 'D:\AutoBackUp\SQLData3\';
DECLARE @DBName NVARCHAR(100);
DECLARE @SQL NVARCHAR(MAX);
DECLARE @LogicalData NVARCHAR(128);
DECLARE @LogicalLog NVARCHAR(128);

-- Liste des bases à restaurer
DECLARE @Databases TABLE (DBName NVARCHAR(100));
INSERT INTO @Databases (DBName) VALUES
('PS_UserData'),
('PS_Billing'),
('PS_GameDefs'),
('PS_GameData'),
('PS_GameLog'),
('PS_ChatLog');

-- Curseur sur les bases
DECLARE db_cursor CURSOR FOR SELECT DBName FROM @Databases;

OPEN db_cursor;
FETCH NEXT FROM db_cursor INTO @DBName;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Supprimer la table temporaire si elle existe
    IF OBJECT_ID('tempdb..#FileList') IS NOT NULL DROP TABLE #FileList;

    -- Créer la table temporaire pour RESTORE FILELISTONLY
    CREATE TABLE #FileList (
        LogicalName NVARCHAR(128),
        PhysicalName NVARCHAR(260),
        Type CHAR(1),
        FileGroupName NVARCHAR(128),
        Size BIGINT,
        MaxSize BIGINT,
        FileID BIGINT,
        CreateLSN NUMERIC(25,0),
        DropLSN NUMERIC(25,0),
        UniqueID UNIQUEIDENTIFIER,
        ReadOnlyLSN NUMERIC(25,0),
        ReadWriteLSN NUMERIC(25,0),
        BackupSizeInBytes BIGINT,
        SourceBlockSize INT,
        FileGroupID INT,
        LogGroupGUID UNIQUEIDENTIFIER,
        DifferentialBaseLSN NUMERIC(25,0),
        DifferentialBaseGUID UNIQUEIDENTIFIER,
        IsReadOnly BIT,
        IsPresent BIT,
        TDEThumbprint VARBINARY(32),
        SnapshotURL NVARCHAR(360)
    );

    -- Extraire les noms logiques depuis le .bak dynamiquement
    SET @SQL = 'INSERT INTO #FileList EXEC(''RESTORE FILELISTONLY FROM DISK = ''''' + @BackupPath + @DBName + '.bak'''''')';
    EXEC(@SQL);

    -- Lire les noms logiques
    SELECT 
        @LogicalData = (SELECT TOP 1 LogicalName FROM #FileList WHERE Type = 'D'),
        @LogicalLog  = (SELECT TOP 1 LogicalName FROM #FileList WHERE Type = 'L');

    -- Afficher les informations utiles
    PRINT '-------------------------------';
    PRINT 'Base à restaurer     : ' + @DBName;
    PRINT 'Fichier source .bak  : ' + @BackupPath + @DBName + '.bak';
    PRINT 'Nom logique (DATA)   : ' + @LogicalData;
    PRINT 'Nom logique (LOG)    : ' + @LogicalLog;
    PRINT 'Cible fichier .mdf   : ' + @DataPath + @DBName + '.mdf';
    PRINT 'Cible fichier .ldf   : ' + @DataPath + @DBName + '_log.ldf';
    PRINT '-------------------------------';

    -- Construire la commande RESTORE finale
    SET @SQL = '
        RESTORE DATABASE [' + @DBName + ']
        FROM DISK = ''' + @BackupPath + @DBName + '.bak''
        WITH 
            MOVE ''' + @LogicalData + ''' TO ''' + @DataPath + @DBName + '.mdf'',
            MOVE ''' + @LogicalLog + ''' TO ''' + @DataPath + @DBName + '_log.ldf'',
            REPLACE;';

    -- Exécuter la commande RESTORE
    EXEC(@SQL);

    -- Passer à la base suivante
    FETCH NEXT FROM db_cursor INTO @DBName;
END

CLOSE db_cursor;
DEALLOCATE db_cursor;
