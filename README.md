# e


Login
view
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib.auth.models import User
from ifrn.views import listar
from django.contrib.auth.decorators import login_required
@login_required
def registro(request):
if request.method == 'POST':
username = request.POST['username']
password = request.POST['password']
password_confirm = request.POST['password_confirm']
if password == password_confirm:
user = User.objects.create_user(username=username,
password=password)
login(request, user)
return redirect('listar')
return render(request, 'registration/registro.html')



TEMPLATES
REGISTRO
user/templates/registration/registro.html
<form method="post">
{% csrf_token %}
<label for="username">Nome de usuário:</label>
<input type="text" id="username" name="username" required>
<label for="password">Senha:</label>
<input type="password" id="password" name="password" required>
<label for="password_confirm">Confirme a senha:</label>
<input type="password" id="password_confirm" name="password_confirm"
required>
<button type="submit">Registrar</button>
</form>


LOGIN
user/templates/registration/login.html
<form method="post">
{% csrf_token %}
{{ form.as_p }}
<button type="submit">Login</button>
</form>



Paginação + Filtro
view
from django.shortcuts import render
from .models import Aluno
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.contrib.auth.decorators import login_required
# Create your views here.
@login_required
def listar(request):
alunos = Aluno.objects.all()
nome = request.GET.get('nome')
aprovado = request.GET.get('aprovado')
if nome:
alunos = alunos.filter(nome=request.GET.get('nome'))
if aprovado =='aprovado':
alunos = alunos.filter(aprovado=True)
if aprovado =='reprovado':
alunos = alunos.filter(aprovado=False)
paginator = Paginator(alunos, 5)
page = request.GET.get('page')
try:
alunos = paginator.page(page)
except PageNotAnInteger:
alunos = paginator.page(1)
except EmptyPage:
alunos = paginator.page(paginator.num_pages)
context = {
'alunos': alunos
}
return render(request, 'ifrn/listar.html', context)






TEMPLATE LISTAR
<form action="{% url 'listar' %}" method="get">
<div>
<input type="text" name="nome">
</div>
<div>
<div>
<input type="radio" name="aprovado" id="aprovado"
value="aprovado">
<label for="aprovado">Aprovado</label>
</div>
<div class="form-check m-1">
<input type="radio" name="aprovado" id="reprovado"
value="reprovado">
<label for="reprovado">Reprovado</label>
</div>
</div>
<div>
<button type="submit">Buscar</button>
<a href="{% url 'listar' %}">Limpar</a>
</div>
</form>
{% for alunos in alunos %}
<ul>
<li> {{alunos.nome}} {{alunos.aprovado}} </li>
</ul>
{% endfor %}
<nav aria-label="Page navigation example">
<ul class="pagination">
{% if alunos.has_previous %}
<li class="page-item">
<a class="page-link" href="?page=1"
aria-label="Primeira">
<span aria-hidden="true">&laquo;&laquo;</span>
</a>
</li>
<li class="page-item">
<a class="page-link" href="?page={{
alunos.previous_page_number }}" aria-label="Anterior">
<span aria-hidden="true">&laquo;</span>
</a>
</li>
{% endif %}
<li class="page-item"><span class="page-link">{{
alunos.number }}</span></li>
{% if alunos.has_next %}
<li class="page-item">
<a class="page-link" href="?page={{
alunos.next_page_number }}" aria-label="Próxima">
<span aria-hidden="true">&raquo;</span>
</a>
</li>
<li class="page-item">
<a class="page-link" href="?page={{
alunos.paginator.num_pages }}" aria-label="Última">
<span aria-hidden="true">&raquo;&raquo;</span>
</a>
</li>
{% endif %}
</ul>
</nav>




urls
from django.contrib import admin
from django.urls import path
from django.contrib.auth import views as auth_views
from ifrn.views import listar
from user.views import registro
urlpatterns = [
path("admin/", admin.site.urls),
path("", listar, name='listar'),
path('registro/', registro, name='registro'),
path('accounts/login/',auth_views.LoginView.as_view(), name='login'),
path('accounts/logout/',auth_views.LogoutView.as_view(),name='logout'),
]
