from flask import Flask, jsonify
import mysql.connector
from mysql.connector import Error
import os
import time

app = Flask(__name__)

def get_db_connection():
    max_retries = 5
    retry_delay = 2
    
    for attempt in range(max_retries):
        try:
            connection = mysql.connector.connect(
                host=os.getenv('DB_HOST', 'mysql'),
                database=os.getenv('DB_NAME', 'miapp_db'),
                user=os.getenv('DB_USER', 'usuario'),
                password=os.getenv('DB_PASSWORD', 'password123')
            )
            if connection.is_connected():
                return connection
        except Error as e:
            print(f"Intento {attempt + 1}/{max_retries} fallido: {e}")
            if attempt < max_retries - 1:
                time.sleep(retry_delay)
            else:
                raise
    return None

@app.route('/')
def hola_mundo():
    try:
        connection = get_db_connection()
        cursor = connection.cursor()
        
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS visitas (
                id INT AUTO_INCREMENT PRIMARY KEY,
                fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # Registrar visita
        cursor.execute("INSERT INTO visitas (fecha) VALUES (NOW())")
        connection.commit()
        
        # Contar visitas
        cursor.execute("SELECT COUNT(*) FROM visitas")
        total_visitas = cursor.fetchone()[0]
        
        cursor.close()
        connection.close()
        
        return jsonify({
            'mensaje': '¡Hola Mundo!',
            'estado': 'Conectado a MySQL exitosamente',
            'total_visitas': total_visitas
        })
        
    except Error as e:
        return jsonify({
            'mensaje': '¡Hola Mundo!',
            'error': f'Error de conexión a MySQL: {str(e)}'
        }), 500

@app.route('/health')
def health():
    """Endpoint de salud"""
    try:
        connection = get_db_connection()
        if connection and connection.is_connected():
            db_info = connection.get_server_info()
            connection.close()
            return jsonify({
                'status': 'healthy',
                'database': 'connected',
                'mysql_version': db_info
            })
    except Error as e:
        return jsonify({
            'status': 'unhealthy',
            'database': 'disconnected',
            'error': str(e)
        }), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
