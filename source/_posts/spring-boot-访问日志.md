---
title: spring boot 访问日志
date: 2021-12-23 13:51:18
tags: java
categories: 语言
---

访问日志在进入controller时才调用，若之前发生错误（如参数转换json对象时，参考spring-boot-原始报文日志），此处无法调用。

```
package com.yier.platform.aop;
 
import com.yier.platform.common.po.medical.res.BaseMedicalRes;
import com.yier.platform.common.util.JsonUtils;
import com.yier.platform.common.util.WebUtil;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
 
@Aspect
@Component
public class VisitLogIntercept {
 
    private Logger logger = LoggerFactory.getLogger(getClass());
 
    private static final int MAX_RESULT_LENGTH = 10000;
 
    /** 打印日志与响应时间 */
    @Around("within(com.yier.platform.controller..*Controller)")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        long curTime = System.currentTimeMillis();
        ServletRequestAttributes req = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        String controller = pjp.getTarget().getClass().getName();
        String method =  pjp.getSignature().getName();
        Object args = WebUtil.getArgs(pjp.getArgs());
 
        if(req == null || req.getRequest() == null){
            return pjp.proceed();
        }
 
        if(args == null && req.getRequest().getParameterMap() != null) {
            args = req.getRequest().getParameterMap();
        }
        logger.info("IP：{}，类：{}，方法：{}，参数：{}", WebUtil.getHost(req.getRequest()), controller, method, JsonUtils.toJsonString(args));
        Object result = pjp.proceed();
 
        // 若返回结果超过1万，则截取前1万个字符打印（实测发现有超过100万的）
        String strResult = "";
        if(result != null) {
            if(result instanceof BaseMedicalRes) {
                strResult = JsonUtils.toJsonString(result);
            } else {
                strResult = result.toString();
            }
            if(strResult.length() > MAX_RESULT_LENGTH) {
                strResult = strResult.substring(0, MAX_RESULT_LENGTH) + "...";
            }
        }
 
        logger.info("类：{}，方法：{}，消耗时间：{}ms，结果：{}", controller, method, System.currentTimeMillis() - curTime, strResult);
        return result;
    }
 
}
```

```
package com.yier.platform.common.util;
 
import com.google.common.collect.Lists;
import org.apache.commons.lang3.StringUtils;
import org.springframework.ui.Model;
import org.springframework.util.CollectionUtils;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.List;
 
/**
 * Web层辅助工具类
 * @author zhimou.qu
 * @create 2018年7月12日
 */
public class WebUtil {
 
    /** 获取当前ServletRequestAttributes */
    public static ServletRequestAttributes getCurServletRequestAttributes() {
        return (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    }
 
    /** 获取当前HttpServletRequest */
    public static HttpServletRequest getCurRequest() {
        return getCurServletRequestAttributes().getRequest();
    }
 
    /** 获取当前HttpServletResponse */
    public static HttpServletResponse getCurResponse() {
        return getCurServletRequestAttributes().getResponse();
    }
 
    /** 获取客户端IP */
    public static final String getHost(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (StringUtils.isBlank(ip) || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (StringUtils.isBlank(ip) || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (StringUtils.isBlank(ip) || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Real-IP");
        }
        if (StringUtils.isBlank(ip) || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        if ("127.0.0.1".equals(ip)) {
            InetAddress inet = null;
            try {
                inet = InetAddress.getLocalHost();
            } catch (UnknownHostException e) {
                e.printStackTrace();
            }
            ip = inet.getHostAddress();
        }
        if (ip != null && ip.length() > 15) {
            if (ip.indexOf(",") > 0) {
                ip = ip.substring(0, ip.indexOf(","));
            }
        }
        return ip;
    }
 
    /**
     * 获取路径
     *
     * @author  wangjf
     * @date    2019年1月24日 下午5:20:22
     * @param request
     * @return
     */
    public static final String getRequestPath(HttpServletRequest request) {
        return request.getServletPath();
    }
 
 
    /**
     * 获取参数
     * @param args
     * @return
     */
    public static final Object getArgs(Object args) {
        if(args != null) {
            if(args instanceof Object[]) {
                List<Object> argList = Lists.newArrayList();
                for(Object object : (Object[])args){
                    if(isServletObject(object)) {
                        continue;
                    }
                    argList.add(object);
                }
                if(CollectionUtils.isEmpty(argList)) {
                    args = null;
                } else {
                    args = argList.toArray();
                }
            } else if(isServletObject(args)) {
                args = null;
            }
        }
        return args;
    }
 
    /**
     * 判断是否是servlet对象
     * @param object
     * @return
     */
    private static boolean isServletObject(Object object) {
        if(object instanceof Model || object instanceof HttpServletRequest || object instanceof ModelAndView || object instanceof MultipartFile) {
            return true;
        }
        return false;
    }
}
```