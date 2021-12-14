run: dependencies
	FLASK_APP=hello.py flask run

tests: dependencies
	./venv/bin/python tests.py

dependencies: venv
	./venv/bin/pip install -r requirements.txt

venv:
	[ -d venv ] || python3 -m venv venv
