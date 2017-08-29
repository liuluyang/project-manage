#coding:utf8
# from django.shortcuts import render_to_response
import urllib2
import json
from datetime import datetime
from django.shortcuts import render,render_to_response,HttpResponseRedirect,HttpResponse
from models import *
import time
import random
from etherpad_lite import EtherpadLiteClient
from dw.models import Project, User, Type
from dw.decorator import login_required
from operator import attrgetter
from dw.permissions import user_permissions

class Pad_api:
    with open('/var/www/etherpad-lite/APIKEY.txt','r') as file_temp:
        apikey = file_temp.read()
    ether_url = 'http://filmind.cn:9001/'
    # ether_url = 'http://192.168.3.133:9001/'
    #返回所有笔记
    def listAllPads(self,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/listAllPads?apikey='+apikey).read())['data']['padIDs']
    #创建笔记
    def createPad(self,padID,ether_url=ether_url,apikey=apikey):
        urllib2.urlopen(ether_url+'api/1.2.13/createPad?apikey='+apikey+'&padID='+padID).read()
    #删除笔记
    def deletePad(self,padID,ether_url=ether_url,apikey=apikey):
        urllib2.urlopen(ether_url+'api/1.2.13/deletePad?apikey='+apikey+'&padID='+padID).read()
    #返回笔记的所有作者
    def listAuthorsOfPad(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/listAuthorsOfPad?apikey='+apikey+'&padID='+padID).read())['data']['authorsID']
    #返回笔记只读ID
    def getReadOnlyID(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/getReadOnlyID?apikey='+apikey+'&padID='+padID).read())['data']['readOnlyID']
    #返回正在使用该笔记的人数
    def padUsersCount(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/padUsersCount?apikey='+apikey+'&padID='+padID).read())['data']['padUsersCount']
    #返回最后一次修改的时间
    def getLastEdited(self,padID,ether_url=ether_url,apikey=apikey):
        return time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(json.loads(urllib2.urlopen(ether_url+'api/1.2.13/getLastEdited?apikey='+apikey+'&padID='+padID).read())['data']['lastEdited']/1000))
    #复制笔记
    def copyPad(self,sourceID,destinationID,ether_url=ether_url,apikey=apikey,force=False):
        urllib2.urlopen(ether_url+'api/1.2.13/copyPadCount?apikey='+apikey+'&sourceID='+sourceID+'&destinationID='+destinationID+'&force='+force).read()
    #移动笔记
    def movePad(self,sourceID,destinationID,ether_url=ether_url,apikey=apikey,force=False):
        urllib2.urlopen(ether_url+'api/1.2.13/movePadCount?apikey='+apikey+'&sourceID='+sourceID+'&destinationID='+destinationID+'&force='+force).read()
    #返回所有正在使用该笔记的用户的信息(colorId,name,timestamp,id)
    def padUsers(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/padUsers?apikey='+apikey+'&padID='+padID).read())['data']['padUsers']
    #返回笔记text
    def getText(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/getText?apikey='+apikey+'&padID='+padID).read())['data']['text']
    #设置笔记text(默认清空)
    def setText(self,padID,text="",ether_url=ether_url,apikey=apikey):
        urllib2.urlopen(ether_url+'api/1.2.13/setText?apikey='+apikey+'&padID='+padID+'&text='+text).read()
    #添加笔记text
    def appendText(self,padID,text="",ether_url=ether_url,apikey=apikey):
        urllib2.urlopen(ether_url+'api/1.2.13/appendText?apikey='+apikey+'&padID='+padID+'&text='+text).read()
    #获取HTML(文本)
    def getHTML(self,padID,ether_url=ether_url,apikey=apikey):
        return json.loads(urllib2.urlopen(ether_url+'api/1.2.13/getHTML?apikey='+apikey+'&padID='+padID).read())['data']['html']
    #设置HTML
    def setHTML(self,padID,html="<!DOCTYPE%20HTML><html><body><br></body></html>",ether_url=ether_url,apikey=apikey):
        urllib2.urlopen(ether_url+'api/1.2.13/setHTML?apikey='+apikey+'&padID='+padID+'&html='+html).read()
    #返回笔记url前缀
    def pad_url(self,ether_url=ether_url):
        return ether_url+'p/'

#笔记列表
@login_required
def show(request,*args,**kwargs):
    url = kwargs['project_id']
    user_now = request.session.get('USERNAME', False)
    user_head_portrait = User.objects.get(username=user_now).head_portrait
    pad_api = Pad_api()

    if request.method == "POST":
        judge = request.POST.get('judge')
        #删除笔记
        if judge == 'pad_delete':
            padID = request.POST.get('padID')
            confirm = request.POST.get('confirm')
            #防止刷新时重复提交报错
            if pad_is_exist_ID(padID):
                u_perm = user_permissions(User.objects.get(username=user_now))
                if 'manage' in u_perm:
                    delete_pad(padID)
                elif Pad_card.objects.get(padID=padID).pad_creator == user_now:
                    if len(pad_api.getText(padID))==1:
                        delete_pad(padID)
                    else:
                        error_message = '该笔记内容不为空,无法完成删除.'
                else:
                    error_message = '您不是管理员或创建者,无权删除该笔记.'

    card_objects = Pad_card.objects.filter(pad_project=url)
    #更新数据
    for card_object in card_objects:
        last_edited_time = pad_api.getLastEdited(card_object.padID)
        Pad_card.objects.filter(padID=card_object.padID).update(pad_change_time=last_edited_time)
    #按修改时间排序
    pad_cards = sorted(card_objects,key=attrgetter('pad_change_time'),reverse=True)
    return render_to_response('show.html', locals())

#删除笔记函数
def delete_pad(padID):
    pad_api = Pad_api()
    padIDs = pad_api.listAllPads()
    pad_objects = Pad_card.objects.all()
    padIDs_card = []
    for pad_object in pad_objects:
        padIDs_card.append(pad_object.padID)
    if padID in padIDs:
        pad_api.deletePad(padID)
    if padID in padIDs_card:
        Pad_card.objects.get(padID=padID).delete()

#创建随机padID
def create_padID():
    pad_api = Pad_api()
    padIDs = pad_api.listAllPads()
    chars = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz_"
    string_length = 20
    random_string = ''
    for i in range(string_length):
        r_num = random.randint(0,len(chars)-1)
        random_string += chars[r_num]
    while random_string in padIDs:
        random_string = ''
        for i in range(string_length):
            r_num = random.randint(0,len(chars)-1)
            random_string += chars[r_num]
    return random_string

#判断笔记是否存在-pad_name
def pad_is_exist_name(pad_name):
    pad_objects = Pad_card.objects.all()
    pad_names = []
    for pad_object in pad_objects:
        pad_names.append(pad_object.pad_name)
    if pad_name in pad_names:
        return True
    else:
        return False

#判断笔记是否存在-padID
def pad_is_exist_ID(padID):
    print 16
    pad_api = Pad_api()
    print 17
    padIDs = pad_api.listAllPads()
    print 18
    print padID
    print padIDs
    if padID in padIDs:
        print 19
        return True
    else:
        print 20
        return False

#创建笔记
@login_required
def create_pad(request,*args,**kwargs):
    url = kwargs['project_id']
    user_now = request.session.get('USERNAME', False)
    user_head_portrait = User.objects.get(username=user_now).head_portrait
    project_object = Project.objects.get(id=url)
    project_type = project_object.project_type
    milestones = project_type.milestone.all()
    prioritys = project_type.task_priority.all()
    statuss = project_type.task_status.all()
    components = project_type.task_component.all()
    types = project_type.task_type.all()
    versions = project_type.task_version.all()
    if request.method == 'POST':
        #获取APIKEY
        with open('/var/www/etherpad-lite/APIKEY.txt','r') as file_temp:
            apikey = file_temp.read()
        #判断笔记是否存在
        name = request.POST.get('pad_name')
        type_id = request.POST.get('type')
        if pad_is_exist_name(name):
            error_message = '记事本已存在，请重新输入。'
            return render_to_response('create_pad.html', locals())
        else:
            #抓取笔记卡片信息
            padID = create_padID()
            pad_name = name
            pad_creator = user_now
            pad_create_time = datetime.now()
            pad_change_time = datetime.now()
            pad_authors = user_now
            pad_project = url
            pad_type = Type.objects.get(id=type_id)
            #保存笔记卡片信息
            pad_card_message = Pad_card(pad_creator=pad_creator,pad_name=pad_name,padID=padID,pad_create_time=pad_create_time,pad_change_time=pad_change_time,pad_authors=pad_authors,pad_project=pad_project,pad_type=pad_type)
            pad_card_message.save()
            #创建笔记
            pad_api = Pad_api()
            padIDs = pad_api.createPad(padID)
            return HttpResponseRedirect('/padlist/'+url+'/show/')
    return render_to_response('create_pad.html',locals())



