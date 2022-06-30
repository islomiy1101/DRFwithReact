## DRF with REACTJS

Loyihani boshlashdan oldin bizga kerakli packagelarni o'rnatib olamiz.

    pip install django djangorestframework
    bu holatda ikkita package birdaniga o'rnaydi.
yangi ochamiz buning uchun esa,

    django-admin startproject APIProject

loyiha ichida yangi app yaratamiz.

    python manage.py startapp api

__api__ appimizni loyihaga bog'laymiz.

    INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'api',

    'rest_framework',
    ]

models.py faylida __Article__ nomli model yaratamiz

    from django.db import models


    class  Article(models.Model):
        title=models.CharField(max_length=250)
        description=models.TextField()
        created_at=models.DateTimeField(auto_now_add=True)


        def __str__(self):
            return self.title

modelimizni bazaga bog'lash uchun quyidagi buyruqlarni terminalga beramiz:

    python manage.py makemigrations
    python manage.py migrate

admin.py fayliga kirib modelimizni admin panelda ko'rinishi uchun quyidagi buyruqlar ketma-ketligini beramiz.

    from django.contrib import admin
    from api.models import Article


    admin.site.register(Article)

__api__ app ichida yangi serializers.py nomli fayl yaratamiz.

    from rest_framework import serializers
    from api.models import Article


    class ArticleSerializer(serializers.ModelSerializer):
        class Meta:
            model=Article
            fields=['id','title','description']

views.py faylida __ArticleViewSet__ nomli class yaratamiz.

    from .models import Article
    from .serializers import ArticleSerializer
    from rest_framework import viewsets

    class ArticleViewSet(viewsets.ModelViewSet):
        queryset = Article.objects.all()
        serializer_class = ArticleSerializer

__api__ app ichida yangi urls.py faylini yaratamiz.

    from django.urls import path,include
    from .views import *
    from rest_framework.routers import DefaultRouter


    router=DefaultRouter()
    router.register('articles', ArticleViewSet, basename='articles')

    urlpatterns=[
        path('api/',include(router.urls))
    ]

endi esa __signUp__ va __login__ qilishni ko'rib chiqamiz.Buning uchun serializers.py faylida __UserSerializer__ nomli class yaratamiz.

    from django.contrib.auth.models import User


    class UserSerializer(serializers.ModelSerializer):

        class Meta:
            model=User
            fields=['id','username','password']

        def create(self, validated_data):
            password = validated_data.pop('password')
            user = super().create(validated_data)
            user.set_password(password)
            user.save()
            return user

Navbat endi permission berishga.settings.py faylida o'zgarish qilamiz

    REST_FRAMEWORK={
        'DEFAULT_PERMISSION_CLASSES':[
            'rest_framework.permissions.AllowAny',
        ]
    }

bu holatda barcha uchun dostup bor bo'ladi apidan foydalanish uchun

    REST_FRAMEWORK={
        'DEFAULT_PERMISSION_CLASSES':[
            'rest_framework.permissions.IsAuthenticated',
        ]
    }
bu holatda esa faqat ro'yxatdan o'tgan va saytga login qilib kirgan foydalanuvchilar uchungina dostup bo'ladi

__Token__ bu narsa bizga dostup olishimiz uchun kerak.Biz ro'yxatn o'tganimizda token beradi va login qilganimizda ushbu tokenni bizga qaytaradi javob tariqasida.
serializers.py faylida Tokenni ulaymiz

    from rest_framework.authtoken.models import Token


    def create(self, validated_data):
        password = validated_data.pop('password')
        user = super().create(validated_data)
        user.set_password(password)
        user.save()
        Token.objects.create(user=user)
        return user

Endi esa login qilish ya'ni authorization bilan shug'ullanishni boshlaymiz.
Dastavval settings.py faylida applar qatoriga quyidagi app ni ulaymiz

    'rest_framework.authtoken',

views.py faylida __UserViewSet__ nomli class yaratamiz.

    class UserViewSet(viewsets.ModelViewSet):

        queryset = User.objects.all()
        serializer_class = UserSerializer

api/urls.py faylida:

    router.register('users', UserViewSet, basename='users')

asosiy urls.py faylida xam quyidagicha o'zgarishlarni amalga oshiramiz

    from rest_framework.authtoken.views import obtain_auth_token


    urlpatterns = [
        path('admin/', admin.site.urls),
        path('',include('api.urls')),
        path('auth/',obtain_auth_token)
    ]

Endi esa Articlelarga faqatgina login qilganidagina ishlashi uchun views.py faylida quyidagicha o'zgarish qilamiz

    from .serializers import ArticleSerializer,UserSerializer
    from rest_framework.authentication import TokenAuthentication
    from rest_framework.permissions import IsAuthenticated


    class ArticleViewSet(viewsets.ModelViewSet):

        queryset = Article.objects.all()
        serializer_class = ArticleSerializer
        permission_classes = [IsAuthenticated]
        authentication_classes = (TokenAuthentication,)

Shu bilan inshaAlloh bizning apimiz tayyor endi Frontend qismiga o'tsak xam bo'ladi ): .

    npx create-react-app . - reactda yangi loyiga ochib olamiz.
    npm install bootstrap  - bootstrap packageini o'rnatib olamiz.

App.js faylida

    import React,{useState,useEffect} from 'react';
    import './App.css';

    function App() {
        const [articles, setArticles] = useState(['First Title','Second Title']);

        return (
            <div className="App">
            <h3>Django Blog</h3>
            {articles.map(article=>{
                return(
                    <h2>{article}</h2>
                )
            })}
            </div>
        );
    }

Yuqorida static holatda ma'lumotlarni chiqarib ko'rdik endi esa navbat api orqali amalga oshiramiz

    function App() {
        const [articles, setArticles] = useState([]);

        useEffect(()=>{
            fetch('http://127.0.0.1:8000/api/articles/',{
            'method':'GET',
            headers:{
                'Content-Type':'application/json',
                'Authorization':'Token 63804db156c23f67a42c62e8a74ba204cf624239'
            }
            })
            .then(resp=>resp.json())
            .then(resp=>setArticles(resp))
            .catch(error=>console.log(error))
        },[])
        return (
            <div className="App">
            <h3>Django Blog</h3>
            {articles.map(article=>{
                return(
                <div key={article.id}>
                    <h5 className='text-success'>{article.title}</h5>
                    <h3>{article.description}</h3>
                </div>
                )
            })}
            </div>
        );
    }

Ko'rib turganimizdek bizga apidan foydalanish uchun ruxsat bo'lmadi.Shuning uchun django-cors-headers packagini o'rantib olamiz.Buning uchun esa terminalga

    pip install django-cors-headers buyrug'ini beramiz
    
settings.py fayliga 

    INSTALLED_APPS = [
        ...,
        "corsheaders",
        ...,
    ]
    
    MIDDLEWARE = [
        ...,
        "corsheaders.middleware.CorsMiddleware",
        "django.middleware.common.CommonMiddleware",
        ...,
    ]
    
    CORS_ALLOWED_ORIGINS = [
        "http://localhost:3000",
    ]
    
src papkasi ichida components nomli papka yaratamiz va uning ichida ArticleList.js nomli fayl yaratamiz.

     import React from 'react'

    const ArticleList = (props) => {
      return (
        <div>
            {props.articles.map(article=>{
                    return(
                    <div key={article.id}>
                        <h5 className='text-success'>{article.title}</h5>
                        <h3>{article.description}</h3>
                    </div>
                    )
                })}
        </div>
      )
    }
    export default ArticleList

App.js fayli

    import ArticleList from './components/ArticleList';

    function App() {
        const [articles, setArticles] = useState([]);
        return (
            <div className="App">
                <h3>Django Blog</h3>
                <ArticleList articles={articles}/> //ArticleList Componentni chaqirib olamiz
            </div>
        );
    }
    export default App;
Agar App.js faylidan articles ni props orqali uzatmaydigan bo'lsak yuqoridagi kabi xatolik chiqadi shuni oldini olish uchun ArticleList.js faylida quyidagicha o'zgarish qilamiz

    {
        props.articles && props.articles.map(article=>{
        return(
        <div key={article.id}>
            <h5 className='text-success'>{article.title}</h5>
            <h3>{article.description}</h3>
        </div>
        )
    })}

Endigi navbat Postlarni __Update__ va __Delete__ qilishga keldi
ArticleList.js fayliga __Update__ va __Delete__ tugmalarini qo'shamiz

    return (
            <div key={article.id}>
              <h5 className="text-success">{article.title}</h5>
              <h3>{article.description}</h3>

              <div className="row mt-3">
                <div className="col-md-1">
                  <button className="btn btn-primary">Update</button>
                </div>
                
                <div className="col-md-1">
                <button className="btn btn-danger">Delete</button>
                </div>
              </div>
              <hr className="bg-warning" />
            </div>
          );
Dastlab __Update__ qilishni ko'rib o'tamiz:

     <button className="btn btn-primary" onClick={()=>editBtn(article)}>Update</button>
     
     const ArticleList = (props) => {

        const editBtn=(article)=>{
            props.editBtn(article);
        }
     }
components papkasida yangi Form.js faylini yaratamiz

App.js faylida 
    
    import Form from './components/Form';
    
    const [editArticle, setEditArticle] = useState(null)
    
    const editBtn=(article)=>{
        setEditArticle(article)
    }
    
    <ArticleList articles={articles} editBtn={editBtn}/>
    <Form article={editArticle}/>
Form.js
    
     return (
        <div>
            {
                props.article && props.article.title
            }
        </div>
      )
============================================================
        import React,{useState} from "react";

        const Form = (props) => {
            const [title, setTitle] = useState(props.article.title)
            const [description, setDescription] = useState(props.article.description)
          return (
            <div>
              {props.article ? (
                <div className="mb-3">
                  <label htmlFor="title"
                  className="form-label">Title</label>
                  <input type="text" id="title" 
                  className="form-control" value={title}
                  placeholder="Enter a title..."/>
                  <label htmlFor="description"
                  className="form-label">Description</label>
                  <textarea id="description" value={description} 
                  className="form-control" rows="5"
                  placeholder="Enter a Description"></textarea>
                  <button className="btn btn-success mt-2">Update Article</button>
                </div>
              ) : null}
            </div>
          );
        };

        export default Form;

App.js faylida errorni bartaraf etish uchun o'zgarish qilamiz

    <div className="App">
            <h3>Django Blog</h3>
            <ArticleList articles={articles} editBtn={editBtn}/>
            {editArticle? <Form article={editArticle}/> : null}
    </div>

Input va Textareaga o'zgarish kiritish uchun quyidagicha o'zgarish qilamiz

     <input type="text" id="title" className="form-control" value={title}
          placeholder="Enter a title..." onChange={(e)=>setTitle(e.target.value)}/>
          
      <textarea id="description" value={description} 
          className="form-control" rows="5" onChange={(e)=>setDescription(e.target.value)}
          placeholder="Enter a Description"></textarea>

Endi Update Article tugmasini bosganda post update bo'lishi uchun tugmaga funksiya qo'shamiz

      <button className="btn btn-success mt-2" onClick={()=>updateArticle(props.article.id)}>
        Update Article
      </button>
      
Yuqorida esa __updateArticle__ nomli funksiya yaratamiz.

     const updateArticle=async(id)=>{
        const config={
            headers:{
                'Content-Type':'application/json',
                'Authorization': `Token 63804db156c23f67a42c62e8a74ba204cf624239`,
              }
        }
        const article=await axios.put(`http://127.0.0.1:8000/api/articles/${id}/`,{
            title,description},config)
            console.log(article);
      }
Form.js va App.js fayllarimizda ba'zi o'zgarishlarni amalga oshiramiz

    {editArticle? <Form article={editArticle} setEditArticle={setEditArticle}/> : null}
    
Form.js
    
    const article=await axios.put(`http://127.0.0.1:8000/api/articles/${id}/`,{
            title,description},config)
    props.setEditArticle({title:'',description:''}) 
    
