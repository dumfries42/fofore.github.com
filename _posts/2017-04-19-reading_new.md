---
layout: post
title: java
categories: [history]
tags: [sotry]
---

'''

void magic(Node *p1, Node *p2){
    bool p1Later = true;
    disp(p1);
    disp(p2);

    Node* tmp = p1;
    while (tmp->next != NULL) {
        tmp = tmp->next;
        if (tmp->data == p2->data){
            p1Later = false;
        }
    }


    Node* delTmp;
    if (p1Later) {

        while (p2->next->data != p1->data) {
            delTmp = p2->next;
            p2->next = p2->next->next;
            delete delTmp;
        }
    } else {
        delTmp = p1;
        while (p1->next->data != p2->data) {
            delTmp = p1->next;
            p1->next = p1->next->next;
            delete delTmp;
        }
    }

}

'''
