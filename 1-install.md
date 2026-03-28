# Step 1: Install python3-venv if not already
sudo apt install python3-full python3-venv -y

# Step 2: Create virtual environment
python3 -m venv venv

# Step 3: Activate it
source venv/bin/activate

# Step 4: Now install requirements (no sudo needed)
pip install -r requirements.txt

# Every time you open a new terminal:
cd ~/openalgo-postgres
source venv/bin/activate
python app.py
