#!/usr/bin/env python3

import os
import ssl
import argparse
import subprocess
import signal
import sys
import base64
from http.server import HTTPServer, SimpleHTTPRequestHandler

port = 4443
server = None
use_auth = False
username = None
password = None

default_dir = '/usr/share/pyserv'
certs_dir = os.path.expanduser('~/.pyserv')
cert_path = os.path.join(certs_dir, 'cert.pem')
key_path = os.path.join(certs_dir, 'key.pem')
pid_file = os.path.join(certs_dir, 'server.pid')
log_file_path = os.path.join(certs_dir, 'server.log')


class AuthHTTPRequestHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if use_auth:
            auth_header = self.headers.get('Authorization')
            if not auth_header:
                self.send_auth_request()
                return
            try:
                auth_decoded = base64.b64decode(auth_header.split(' ')[1]).decode('utf-8')
                user, passwd = auth_decoded.split(':', 1)
                if user != username or passwd != password:
                    raise ValueError
            except Exception:
                self.send_auth_request()
                return
        super().do_GET()

    def send_auth_request(self):
        self.send_response(401)
        self.send_header('WWW-Authenticate', 'Basic realm="Secure Area"')
        self.end_headers()
        self.wfile.write(b'Authentication required.')

    def log_message(self, format, *args):
        with open(log_file_path, 'a') as log_file:
            log_file.write("%s - - [%s] %s\n" % (
                self.client_address[0],
                self.log_date_time_string(),
                format % args
            ))


def start_server(directory):
    class CustomHandler(AuthHTTPRequestHandler):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, directory=directory, **kwargs)

    os.makedirs(certs_dir, exist_ok=True)
    server = HTTPServer(('0.0.0.0', port), CustomHandler)
    ssl_context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
    ssl_context.load_cert_chain(certfile=cert_path, keyfile=key_path)
    server.socket = ssl_context.wrap_socket(server.socket, server_side=True)

    with open(pid_file, 'w') as f:
        f.write(str(os.getpid()))

    server.serve_forever()


def stop_server():
    if os.path.exists(pid_file):
        with open(pid_file, 'r') as f:
            pid = int(f.read())
        try:
            os.kill(pid, signal.SIGTERM)
            print(f"Server with PID {pid} stopped.")
        except ProcessLookupError:
            print("Process not found.")
        os.remove(pid_file)
    else:
        print("No PID file found. Server might not be running.")


def generate_certs():
    os.makedirs(certs_dir, exist_ok=True)
    if not os.path.exists(cert_path) or not os.path.exists(key_path):
        subprocess.run([
            'openssl', 'req', '-nodes', '-x509',
            '-newkey', 'rsa:4096',
            '-keyout', key_path,
            '-out', cert_path,
            '-days', '365',
            '-subj', '/CN=mylocalhost'
        ], check=True)
        print("Certificates generated in ~/.pyserv/")


def validate_directory(directory):
    if not os.path.isdir(directory):
        print(f"Error: Directory '{directory}' does not exist.")
        sys.exit(1)


def main():
    global use_auth, username, password

    parser = argparse.ArgumentParser(
        description='Expose files via HTTPS.',
        epilog='Examples:\n'
               ' pyserv start -d CTF --passwd admin:1234\n'
               ' pyserv start -d /home/scripts --nopass\n'
               ' pyserv start --passwd 1234\n'
               ' pyserv status\n'
               ' pyserv stop',
        formatter_class=argparse.RawTextHelpFormatter)

    subparsers = parser.add_subparsers(dest='command', required=True)

    # Public commands
    start_parser = subparsers.add_parser('start', help='Start the server')
    start_parser.add_argument('-d', '--directory', default=default_dir,
                              help='Directory to serve (default: %(default)s)')
    auth_group = start_parser.add_mutually_exclusive_group()
    auth_group.add_argument('--nopass', action='store_true', help='No authentication')
    auth_group.add_argument('--passwd', help='Set password (format: username:password or just password)')

    subparsers.add_parser('stop', help='Stop the server')
    subparsers.add_parser('status', help='Check if server is running')

    # Hidden internal command
    internal_parser = subparsers.add_parser('_internal', help=argparse.SUPPRESS)
    internal_parser.add_argument('-d', '--directory', required=True)
    internal_parser.add_argument('--nopass', action='store_true')
    internal_parser.add_argument('--passwd')

    # Remove _internal from help by removing from parser._subparsers._actions
    # This prevents it from appearing in help but keeps it functional
    for action in parser._actions:
        if isinstance(action, argparse._SubParsersAction):
            # Filter out _internal parser from choices for help display only
            # But keep it in choices to keep functionality
            action._choices_actions = [a for a in action._choices_actions if a.dest != '_internal']
            break

    args = parser.parse_args()

    if args.command == 'start':
        directory = args.directory
        validate_directory(directory)
        generate_certs()

        if args.passwd:
            use_auth = True
            if ':' in args.passwd:
                username, password = args.passwd.split(':', 1)
            else:
                username = 'admin'
                password = args.passwd
        else:
            use_auth = False

        print(f"Server running on https://0.0.0.0:{port}")
        print(f"Serving directory: {directory}")
        print("Mode:", "with password" if use_auth else "without password")
        if use_auth:
            print(f"Auth username: {username}, password: {'*' * len(password)}")

        cmd = [
            sys.executable, __file__, '_internal',
            '-d', directory
        ]
        if use_auth:
            cmd += ['--passwd', f"{username}:{password}"]
        else:
            cmd.append('--nopass')

        with open(log_file_path, 'a') as log_file:
            subprocess.Popen(cmd, stdout=log_file, stderr=log_file)

    elif args.command == '_internal':
        directory = args.directory
        if args.passwd:
            use_auth = True
            if ':' in args.passwd:
                username, password = args.passwd.split(':', 1)
            else:
                username = 'admin'
                password = args.passwd
        else:
            use_auth = False
        start_server(directory)

    elif args.command == 'stop':
        stop_server()

    elif args.command == 'status':
        if os.path.exists(pid_file):
            with open(pid_file, 'r') as f:
                print(f"Server is running with PID {f.read().strip()}")
        else:
            print("Server is not running.")


if __name__ == '__main__':
    main()
