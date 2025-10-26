# main.py
import os
import threading
import time
import requests
from flask import Flask, jsonify

app = Flask(_name_)

TARGET_URL = os.getenv("TARGET_URL", "https://www.websurf.cz/auto/?name=dobkarol@gmail.com")
INTERVAL = int(os.getenv("INTERVAL_SECONDS", "60"))
REQUEST_TIMEOUT = int(os.getenv("REQUEST_TIMEOUT", "10"))

recent_logs = []

def background_task():
    while True:
        ts = time.strftime("%Y-%m-%d %H:%M:%S", time.gmtime())
        try:
            r = requests.get(TARGET_URL, timeout=REQUEST_TIMEOUT)
            log = {"time": ts, "status": r.status_code, "len": len(r.content)}
        except Exception as e:
            log = {"time": ts, "error": str(e)}
        recent_logs.insert(0, log)
        if len(recent_logs) > 50:
            recent_logs.pop()
        print("[keepalive]", log)
        time.sleep(max(1, INTERVAL))

thread = threading.Thread(target=background_task, daemon=True)
thread.start()

@app.route("/")
def index():
    return jsonify({"message":"running","target":TARGET_URL,"interval":INTERVAL,"recent":recent_logs[:10]})

if _name_ == "_main_":
    port = int(os.getenv("PORT", "10000"))
    app.run(host="0.0.0.0", port=port)
