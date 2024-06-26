# Customer-s-complaint-
base,html <!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="viewport" content="width=device-width, initial-scale=1.0"><title>{% block title %}{% endblock %}</title><link rel="icon" type="image" href="{{ url_for('static', filename='images/cart logo white-modified.png') }}"><!-- Linking css, js, Google fonts --><link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin><link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}"/><linkhref="https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap" rel="stylesheet"><script src="{{ url_for('static', filename='js/pass.js') }}"></script><link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"><!-- Linking Watson Assistant -->{% block watson %}{% endblock %} </head><body>{% block alert %}{% if to_show %} <script>alert('{{ message }}') </script>{% endif %}{% endblock %}{% block main %}{% endblock %} </body></html>login.html:{% extends 'base.html' %}{% block title %}Login{% endblock %}{% block main %} <div class="bg-main-div"><section class="login-section"><div class="login-div"><div class="login-header"><img src="{{ url_for('static', filename='images/cart logo white.png') }}" class="login-img" alt="logo" /><h2>Sign in</h2><p>Use your Registry Account</p></div><div class="login-remind"><form action="{{ url_for('blue_print.login') }}" method="POST" class="login-form"><label>Email</label><input type="email" required value="{{ email }}" name="email" placeholder="Enter your email"/><label>Password</label><input type="password" required value="{{ password }}" name="password" id="password-input" placeholder="Enter your password"/><div class="show-pass-div"><input type="checkbox" onclick="showPassword()" style="height: 20px;"/><p>Show Password</p></div><div class="role-div"><p>Role : </p><div><div><input type="radio" style="height: 20px;" value="Customer" checked name="role-check"/><p>Customer</p></div><div><input type="radio" style="height: 20px;" value="Agent" name="role-check"/><p>Agent</p></div></div></div><button class="submit-btn" type="submit">Login</button><div><!-- {{ url_for('blue_print.forgot') }} --><a href="{{ url_for('blue_print.forgot') }}" class="links">Forgot Password?</a> <br><div><a href="{{ url_for('blue_print.register') }}" class="links">Don't have an account yet? Register</a></div></div></form></div></div></section></div>{% endblock %}address.html:{% extends 'base.html' %}{% block title %}AddressColumn{% endblock %}{% block main %} <div class="dashboard-div"><nav><div class="dash-nav"><div><div class="dash-img-text">{% if user == "AGENT" %} <a href="{{ url_for('agent.assigned') }}"><i class="fa fa-arrow-left" aria-hidden="true"></i></a><img src="{{ url_for('static', filename='images/cust profile.png') }}" class="img-in-nav" alt="logo"/>{% else %} <a href="{{ url_for('customer.tickets') }}"><i class="fa fa-arrow-left" aria-hidden="true"></i></a><img src="{{ url_for('static', filename='images/agent.png') }}" class="img-in-nav" alt="logo"/>{% endif %} <h3>{{ name }}</h3></div></div><div><div style="align-items: center;">{% if value == "True" %}{% if user == "CUSTOMER" %} <a href="/customer/close/{{ id }}"><button class="logout-btn">CLOSE TICKET</button></a>{% endif %}{% endif %} </div></div></div></nav><div class="chat-body"><div class="chat-contents" id="content">{% if msgs_to_show %}{% for chat in chats %}{% if chat['SENDER_ID'] == sender_id %} <div class="message-sent">{{ chat['MESSAGE'] }}</div>{% else %} <div class="message-sent received">{{ chat['MESSAGE'] }}</div>{% endif %}{% endfor %}{% endif %} </div><div class="chat-input-div">{% if value == "True" %} <form method="POST" action="{{ post_url }}"><input name="message-box" class="chat-input" type="text" placeholder="Type something" required/><button type="submit" class="chat-send"><i class="fa fa-paper-plane-o" aria-hidden="true"></i></button></form>{% else %} <div>{% if user == "CUSTOMER" %}<h4>You closed this ticket. Chats are disabled</h4>{% else %} <h4>{{ name }} closed this ticket. Chats are disabled</h4>{% endif %} </div>{% endif %} </div></div></div>{% endblock %}chat.py:from flask import render_template, Blueprint, request, session, redirect, url_forimport ibm_dbfrom datetime import datetimeimport timechat = Blueprint("chat_bp", name )@chat.route('/chat/<ticket_id>/<receiver_name>/', methods = ['GET', 'POST'])def address(ticket_id, receiver_name):''' Address Column - Agent and Customer chats with one another''': param ticket_id ID of the ticket for which the chat is being opened: param receiver_name Name of the one who receives the texts, may be Agent / Customer # common page for both the customer and the agent# so cannot use login_required annotation# so to know who signed in, we have to use the sessionuser = "" sender_id = "" value = "" can_trust = Falsepost_url = f'/chat/{ticket_id}/{receiver_name}/'if session['LOGGED_IN_AS'] is not None:if session['LOGGED_IN_AS'] == "CUSTOMER": # checking if the customer is really logged in# by checking, if the customer has uuid attributefrom .views import customerif(hasattr(customer, 'uuid')): user = "CUSTOMER" sender_id = customer.uuidcan_trust = Trueelse: # logging out the so called customerreturn redirect(url_for('blue_print.logout')) elif session['LOGGED_IN_AS'] == "AGENT": # checking if the agent is really logged in# by checking, if the agent has uuid aatributefrom .views import agentif (hasattr(agent, 'uuid')): user = "AGENT" sender_id = agent.uuidcan_trust = Trueelse: # Admin is the one who logged in# admin should not see the chats, sp directly logging the admin out return redirect(url_for('blue_print.logout'))to_show = False message = ""if can_trust: # importing the connection stringfrom .views import connif request.method == 'POST': # chats are enabled, only if the ticket is OPEN# getting the data collected from the customer / agent myMessage = request.form.get('message-box')if len(myMessage) == 0:to_show = True message = "Type something!"else: # inserting the message in the database# query to insert the message in the database message_insert_query = ''' INSERT INTO chat(chat_id, sender_id, message, sent_at)VALUES(?, ?, ?, ?)'''try:stmt = ibm_db.prepare(conn, message_insert_query)ibm_db.bind_param(stmt, 1, ticket_id)ibm_db.bind_param(stmt, 2, sender_id)ibm_db.bind_param(stmt, 3, myMessage)ibm_db.bind_param(stmt, 4, datetime.now())ibm_db.execute(stmt) except:to_show = True message = "Please send again!"return redirect(post_url) else: # method is GET# retrieving all the messages, if exist from the database msgs_to_show = False# query to get all the messages for this ticketget_messages_query = ''' SELECT * FROM chatWHERE chat_id = ?ORDER BY sent_at ASC''' # query to check if the ticket is still OPENquery_status_check = '''SELECT query_status FROM tickets WHERE ticket_id = ?'''try:# first checking if the ticket is OPENcheck = ibm_db.prepare(conn, query_status_check)ibm_db.bind_param(check, 1, ticket_id)ibm_db.execute(check) value = "True" if ibm_db.fetch_assoc(check)['QUERY_STATUS'] == "OPEN" else "False" # getting all the messages concerned with this ticket stmt = ibm_db.prepare(conn, get_messages_query)ibm_db.bind_param(stmt, 1, ticket_id)ibm_db.execute(stmt) messages = ibm_db.fetch_assoc(stmt) messages_list = [] while messages != False: messages_list.append(messages)print(messages) messages = ibm_db.fetch_assoc(stmt)# then some messages exist in this chatif len(messages_list) > 0: msgs_to_show = Trueelif len(messages_list) == 0 and value == "True": # ticket is OPEN# but no messages are sent b/w the customer and the agent msgs_to_show = Falseto_show = True message = f'Start the conversation with the {"Customer" if user == "AGENT" else "Agent"}' except:to_show = True message = "Something happened! Try Again"return render_template('address.html',to_show = to_show, message = message,id = ticket_id, chats = messages_list, msgs_to_show = msgs_to_show, sender_id = sender_id, name = receiver_name, user = user, post_url = post_url, value = value) else: # logging out whoever came inside the linkreturn redirect(url_for('blue_print.logout'), user = user)init .py:from flask import Flask, sessionfrom flask_login import LoginManager def create_app(): app = Flask( name ) app.config['SECRET_KEY'] = "PHqtYfAN2v@CCR2022" # registering the blue prints with the appfrom .routes.views import views app.register_blueprint(views, appendix='/')from .routes.cust import cust app.register_blueprint(cust, appendix='/customer/')from .routes.admin import adminapp.register_blueprint(admin, appendix='/admin/')from .routes.agent import agent app.register_blueprint(agent, appendix='/agent/')from .routes.chat import chat app.register_blueprint(chat, appendix='/chat/')# setting up the login managerlogin_manager = LoginManager()login_manager.login_view = "blue_print.login"login_manager.init_app(app)@login_manager.user_loader def load_user(id):if session.get('LOGGED_IN_AS') is not None:if session['LOGGED_IN_AS'] == "CUSTOMER":from .routes.views import customerif hasattr(customer, 'first_name'): return customer elif session['LOGGED_IN_AS'] == "AGENT":from .routes.views import agentif hasattr(agent, 'first_name'): return agent elif session['LOGGED_IN_AS'] == "ADMIN":from .routes.views import adminif hasattr(admin, 'email'): return adminelse: return Nonereturnapp
