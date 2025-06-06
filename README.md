Si ce projet vous a aidé, pensez à lui laisser une étoile (⭐) en haut à droite

# SQL-RESTOR-AUTOMATIC
A dynamic SQL Server script to automatically restore multiple databases from .bak files using RESTORE FILELISTONLY. Avoids manual logical file name mapping. Ideal for dev, QA, or CI/CD environments.


# SQL Server – Automated Database Restore Script

This script restores multiple SQL Server databases from `.bak` files using dynamic detection of logical file names.

## Features

- Restore multiple databases in a loop
- Automatically detects logical file names
- Uses `MOVE` and `REPLACE`
- Outputs detailed info for each step

## Usage

1. Set the `@BackupPath` and `@DataPath`
2. List your database names in `@Databases`
3. Run the script in SQL Server Management Studio (SSMS)

## License

MIT License — free to use, modify, distribute. Attribution appreciated.
